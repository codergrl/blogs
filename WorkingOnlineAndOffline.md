# Don't repeat yourself! Using extensions to work Online and Offline with the .NET Runtime SDK

Building an app that works in a disconnected environment may seem like a daunting task, but it doesn't have to be. The ArcGIS Runtime SDK offers the same type of functionality (view, query, add, edit, delete) on both online and offline data. It sometimes looks a little differently between online and offline, but that can easily be reconciled with the use of [extension methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods).

## Problem

For example, when you're working with an online feature layer and you're trying to retrieve its related records, you could do something like this:

```csharp
var table = feature.FeatureTable as ServiceFeatureTable;
var relatedTablesList = table.GetRelatedTables();
```

In offline mode, the table is cast to a GeodatabaseFeatureTable:

```csharp
var table = feature.FeatureTable as GeodatabaseFeatureTable;
var relatedTablesList = table.GetRelatedTables();
```

That's not that much different. However, if you want to retrieve the related records for a specific relationship and you want to make sure all the related records are returned, both the table type and the parameters look different in online vs. offline modes. Note: here we use the first relationship but you can loop through them to select which one you prefer.

### Online

```csharp
var table = feature.FeatureTable as ServiceFeatureTable;
var relationshipInfo = table.LayerInfo.RelationshipInfos.FirstOrDefault();
var parameters = new RelatedQueryParameters(relationshipInfo);
var relationships = await (table).QueryRelatedFeaturesAsync(
                                        (ArcGISFeature)feature,
                                        parameters,
                                        QueryFeatureFields.LoadAll);
```

### Offline

```csharp
var table = feature.FeatureTable as GeodatabaseFeatureTable;
var relationshipInfo = table.LayerInfo.RelationshipInfos.FirstOrDefault();
var parameters = new RelatedQueryParameters(relationshipInfo);
var relationships = await (table).QueryRelatedFeaturesAsync(
                                        (ArcGISFeature)feature,
                                        parameters);
```

Since C# is type-safe, it is at a disadvantage when it comes to specifying types that share common functionality. Ideally you could do something like this and move on with life:

```csharp
var table = (AppState == AppState.Online) ?
            (ServiceFeatureTable)feature.FeatureTable :
            (GeodatabaseFeatureTable)feature.FeatureTable;
```

But you cannot. You'll get this error:

`Type of conditional expression cannot be determined because there is no implicit conversion between 'Esri.ArcGISRuntime.Data.ServiceFeatureTable' and 'Esri.ArcGISRuntime.Data.GeodatabaseFeatureTable'.`

## Solution

So what is a solution that will keep you from having to duplicate code? How about a set of [extension methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) that define all the methods you're needing to use in both online and offline mode:

```csharp
/// <summary>
/// Retrieves all relationship infos for a table
/// </summary>
internal static IReadOnlyList<RelationshipInfo> GetRelationshipInfos(this FeatureTable featureTable)
{
    if (featureTable is ServiceFeatureTable serviceFeatureTable)
    {
        return serviceFeatureTable.LayerInfo.RelationshipInfos;
    }
    else if (featureTable is GeodatabaseFeatureTable geodatabaseFeatureTable)
    {
        return geodatabaseFeatureTable.LayerInfo.RelationshipInfos;
    }
    return null;
}

/// <summary>
/// Retrieves records related to a feature based on information from the relationship info
/// </summary>
internal static async Task<IReadOnlyList<RelatedFeatureQueryResult>> GetRelatedRecords(this FeatureTable featureTable, Feature feature, RelationshipInfo relationshipInfo)
{
    var parameters = new RelatedQueryParameters(relationshipInfo);

    if (featureTable is ServiceFeatureTable serviceFeatureTable)
    {
        return await serviceFeatureTable.QueryRelatedFeaturesAsync((ArcGISFeature)feature, parameters, QueryFeatureFields.LoadAll);
    }
    else if (featureTable is GeodatabaseFeatureTable geodatabaseFeatureTable)
    {
        return await geodatabaseFeatureTable.QueryRelatedFeaturesAsync((ArcGISFeature)feature, parameters);
    }
    return null;
}
```

Once you have it all set up, the new extension methods will be available on the base FeatureTable class.

```csharp
// get RelationshipInfos from the table
var relationshipInfos = feature.FeatureTable.GetRelationshipInfos();

// get related records for a feature
var relatedRecords = await feature.FeatureTable.GetRelatedRecords(feature, relationshipInfo);
```

To see this workflow used in a real application, check out the open source [Data Collection](https://github.com/Esri/data-collection-dotnet) app.
