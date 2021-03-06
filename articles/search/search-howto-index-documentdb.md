---
title: "Indexación de un origen de datos de DocumentDB para Azure Search| Microsoft Docs"
description: "En este artículo se muestra cómo crear el indexador de Azure Search con DocumentDB como origen de datos."
services: search
documentationcenter: 
author: chaosrealm
manager: pablocas
editor: 
ms.assetid: 
ms.service: documentdb
ms.devlang: rest-api
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: search
ms.date: 04/11/2017
ms.author: eugenesh
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: 5f657ed128103d4bf1304dfc5fae8d86ef950d87
ms.lasthandoff: 04/12/2017


---
# <a name="connecting-documentdb-with-azure-search-using-indexers"></a>Conexión de DocumentDB con Azure Search mediante indizadores

Si desea implementar una excelente función de búsqueda en los datos de DocumentDB, puede utilizar un indexador de Azure Search para extraer datos en un índice de Azure Search. En este artículo mostramos cómo integrar Azure DocumentDB con Azure Search sin necesidad de escribir código alguno para mantener la infraestructura de indexación.

Para configurar un indexador de DocumentDB, es preciso tener un [servicio Azure Search](search-create-service-portal.md) y crear un índice, un origen de datos y, por último, el indexador. Para crear estos objetos, se pueden usar el [portal](search-import-data-portal.md), el [SDK de .NET](/dotnet/api/microsoft.azure.search) o la [API de REST](/rest/api/searchservice/) en todos los lenguajes que no sean de .NET. 

Si opta por la primera opción, el [Asistente para la importación de datos ](search-import-data-portal.md) lo guía a través de la creación de estos recursos.

> [!NOTE]
> Puede iniciar el asistente para **importar datos** desde el panel de DocumentDB para simplificar la indexación para ese origen de datos. En el panel de navegación de la izquierda, vaya a **Colecciones** > **Add Azure Search** (Añadir búsqueda de Azure) para empezar.

<a name="Concepts"></a>
## <a name="azure-search-indexer-concepts"></a>Conceptos del indexador de Azure Search
Azure Search admite la creación y administración de orígenes de datos (incluida DocumentDB), así como de indizadores que se usan en dichos orígenes.

A **origen de datos** especifica los datos para indexar, las credenciales y las directivas para identificar cambios en los datos (por ejemplo, los documentos modificados o eliminados dentro de la colección). El origen de datos se define como un recurso independiente para que puedan usarlo múltiples indizadores.

Un **indizador** describe cómo fluyen los datos desde el origen de datos a un índice de búsqueda de destino. Los indexadores se pueden utilizar para:

* Realizar una copia única de los datos para rellenar un índice.
* Sincronizar un índice con los cambios del origen de datos en una programación. La programación forma parte de la definición del indizador.
* Invocar actualizaciones a petición para un índice según sea necesario.

<a name="CreateDataSource"></a>
## <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos
Pasos para crear un origen de datos, realice un POST:

    POST https://[service name].search.windows.net/datasources?api-version=2016-09-01
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mydocdbdatasource",
        "type": "documentdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myDocDbEndpoint.documents.azure.com;AccountKey=myDocDbAuthKey;Database=myDocDbDatabaseId"
        },
        "container": { "name": "myDocDbCollectionId", "query": null },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        }
    }

El cuerpo de la solicitud contiene la definición del origen de datos, que debe incluir los siguientes campos:

* **nombre**: elija un nombre para representar la base de datos de DocumentDB.
* **type**: debe ser `documentdb`.
* **credenciales**:
  
  * **connectionString**: obligatorio. Especifique la información de conexión a Azure DocumentDB con el formato siguiente: `AccountEndpoint=<DocumentDB endpoint url>;AccountKey=<DocumentDB auth key>;Database=<DocumentDB database id>`
* **contenedor**:
  
  * **nombre**: obligatorio. Especifique el identificador de la colección de DocumentDB que se va a indexar.
  * **consulta**: opcional. Puede especificar una consulta para acoplar un documento JSON arbitrario en un esquema plano que Azure Search pueda indizar.
* **dataChangeDetectionPolicy**: se recomienda. Consulte la sección [Indexación de documentos modificados](#DataChangeDetectionPolicy).
* **dataDeletionDetectionPolicy**: opcional. Consulte la sección [Indexación de documentos eliminados](#DataDeletionDetectionPolicy).

### <a name="using-queries-to-shape-indexed-data"></a>Uso de consultas para dar forma a los datos indizados
Puede especificar una consulta de DocumentDB para eliminar el formato de las propiedades o matrices anidadas y de las propiedades de JSON del proyecto, y para filtrar los datos que se van a indizar. 

Documento de ejemplo:

    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "documentdb", "search"]
    }

Consulta de filtro:

    SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark

Consulta sin formato:

    SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark
    
    
Consulta de proyección:

    SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark


Consulta sin formato de matriz:

    SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark

<a name="CreateIndex"></a>
## <a name="step-2-create-an-index"></a>Paso 2: Creación de un índice
Si aún no tiene un índice de Azure Search de destino, créelo. Puede crearlo mediante la [interfaz de usuario de Azure Portal](search-create-index-portal.md), la [API de REST de Create Index](/rest/api/searchservice/create-index) o la [clase Index](/dotnet/api/microsoft.azure.search.models.index).

En el ejemplo siguiente se crea un índice con un campo de identificador y de descripción:

    POST https://[service name].search.windows.net/indexes?api-version=2016-09-01
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }

Asegúrese de que el esquema del índice de destino es compatible con el de los documentos JSON de origen o el resultado de la proyección de consultas personalizada.

> [!NOTE]
> Para las colecciones con particiones, la clave de documento predeterminada es la propiedad `_rid` de DocumentDB, que cambia su nombre a `rid` en Azure Search. Además, los valores `_rid` de DocumentDB contienen caracteres que no son válidos en las claves de Azure Search. Por este motivo, la `_rid` valores están codificados con Base64.
> 
> 

### <a name="mapping-between-json-data-types-and-azure-search-data-types"></a>asignación entre tipos de datos de JSON y de Azure Search
| TIPO DE DATOS DE JSON | TIPOS DE CAMPOS DE ÍNDICE DE DESTINO COMPATIBLES |
| --- | --- |
| Booleano |Edm.Boolean, Edm.String |
| Números que parecen enteros |Edm.Int32, Edm.Int64, Edm.String |
| Números que parecen puntos flotantes |Edm.Double, Edm.String |
| Cadena |Edm.String |
| Matrices de tipos primitivos, por ejemplo ["a", "b", "c"] |Collection(Edm.String) |
| Cadenas que parecen fechas |Edm.DateTimeOffset, Edm.String |
| Objetos GeoJSON, por ejemplo, {"tipo": "Punto", "coordenadas": [long, lat]} |Edm.GeographyPoint |
| Otros objetos JSON |N/D |

<a name="CreateIndexer"></a>
## <a name="step-3-create-an-indexer"></a>Paso 3: Creación de un indexador

Una vez creados el origen de datos y los índices, ya podrá crear el indizador:

    POST https://[service name].search.windows.net/indexers?api-version=2016-09-01
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mydocdbindexer",
      "dataSourceName" : "mydocdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }

Este indexador se ejecuta cada dos horas (el intervalo de programación se establece en "PT2H"). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de 5 minutos. La programación es opcional: si se omite, el indizador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API de creación de indexador, consulte [Crear indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer).

<a id="RunIndexer"></a>
### <a name="running-indexer-on-demand"></a>Ejecución del indexador a petición
Además de ejecutarse periódicamente de forma programada, un indexador también puede invocarse a petición:

    POST https://[service name].search.windows.net/indexers/[indexer name]/run?api-version=2016-09-01
    api-key: [Search service admin key]

> [!NOTE]
> Cuando la API de Run se devuelve correctamente, se ha programado la invocación del indexador, pero el procesamiento real se produce de forma asincrónica. 

El estado del indexador se puede supervisar en el portal o mediante la API de Get Indexer Status, lo que se describe a continuación. 

<a name="GetIndexerStatus"></a>
### <a name="getting-indexer-status"></a>Obtención del estado del indexador
Puede recuperar el estado y el historial de ejecución de un indexador:

    GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2016-09-01
    api-key: [Search service admin key]

La respuesta contiene el estado general del indexador, la última vez que se invocó al indexador (o en curso) y el historial de las recientes invocaciones al indexador.

    {
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
         },
        "executionHistory":[ {
            "status":"success",
             "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        }]
    }

El historial de ejecución contiene como máximo las 50 ejecuciones completadas más recientemente en orden cronológico inverso (la ejecución más reciente aparece en primer lugar).

<a name="DataChangeDetectionPolicy"></a>
## <a name="indexing-changed-documents"></a>Indexación de documentos modificados
El fin de una directiva de detección de cambios de datos es identificar de forma eficaz los elementos de datos que han cambiado. Actualmente, `High Water Mark` es la única directiva compatible que usa la propiedad `_ts` (marca de hora) que proporciona DocumentDB, que se especifica de la siguiente forma:

    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }

Se recomienda encarecidamente usar esta directiva para garantizar un buen rendimiento del indexador. 

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad `_ts`. 

<a name="DataDeletionDetectionPolicy"></a>
## <a name="indexing-deleted-documents"></a>Indexación de documentos eliminados
Cuando se eliminan filas de la recopilación, lo habitual es que también se deseen eliminar del índice de búsqueda. El fin de una directiva de detección de eliminación de datos es identificar eficazmente los elementos de datos eliminados. Actualmente, solo es compatible la directiva `Soft Delete` (la eliminación se indica con algún tipo de marca), especificada de la siguiente forma:

    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad a la que `softDeleteColumnName` hace referencia.

En el ejemplo siguiente se crea un origen de datos con una directiva de eliminación temporal:

    POST https://[Search service name].search.windows.net/datasources?api-version=2016-09-01
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mydocdbdatasource",
        "type": "documentdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myDocDbEndpoint.documents.azure.com;AccountKey=myDocDbAuthKey;Database=myDocDbDatabaseId"
        },
        "container": { "name": "myDocDbCollectionId" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }

## <a name="NextSteps"></a>Pasos siguientes
¡Enhorabuena! En este artículo ha aprendido a integrar Azure DocumentDB con Azure Search mediante el indexador de DocumentDB.

* Para más información sobre Azure DocumentDB, vea la [página del servicio DocumentDB](https://azure.microsoft.com/services/documentdb/).
* Para más información sobre Azure Search, vea la [página del servicio Search](https://azure.microsoft.com/services/search/).
