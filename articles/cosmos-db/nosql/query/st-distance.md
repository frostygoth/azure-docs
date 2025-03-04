---
title: ST_DISTANCE in Azure Cosmos DB query language
description: Learn about SQL system function ST_DISTANCE in Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: nosql
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: girobins
ms.custom: query-reference, ignite-2022
---
# ST_DISTANCE (Azure Cosmos DB)
[!INCLUDE[NoSQL](../../includes/appliesto-nosql.md)]

 Returns the distance between the two GeoJSON Point, Polygon, MultiPolygon or LineString expressions. To learn more, see the [Geospatial and GeoJSON location data](geospatial-intro.md) article.
  
## Syntax
  
```sql
ST_DISTANCE (<spatial_expr>, <spatial_expr>)  
```  
  
## Arguments
  
*spatial_expr*  
   Is any valid GeoJSON Point, Polygon, or LineString object expression.  
  
## Return types
  
  Returns a numeric expression containing the distance. This is expressed in meters for the default reference system.  
  
## Examples
  
  The following example shows how to return all family documents that are within 30 km of the specified location using the `ST_DISTANCE` built-in function. 
  
```sql
SELECT f.id
FROM Families f
WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000  
```  
  
 Here is the result set.  
  
```json
[{  
  "id": "WakefieldFamily"  
}]  
```

## Remarks

This system function will benefit from a [geospatial index](../../index-policy.md#spatial-indexes) except in queries with aggregates.

> [!NOTE]
> The GeoJSON specification requires that points within a Polygon be specified in counter-clockwise order. A Polygon specified in clockwise order represents the inverse of the region within it.

## Next steps

- [System functions Azure Cosmos DB](system-functions.yml)
- [Introduction to Azure Cosmos DB](../../introduction.md)
