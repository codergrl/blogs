# Don't duplicate code when working Online and Offline with the .NET Runtime SDK

Building an app that works in a disconnected environment may seem like a daunting task, but it doesn't have to be. The ArcGIS Runtime SDK offers the same type of functionality (view, query, add, edit, delete) on both online and offline data. It sometimes looks a little differently between online and offline, but that can easily be reconciled with the use of an interface. 

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

So what is a solution that will keep you from having to duplicate code? How about an interface that defines all the methods you're needing to use in both online and offline mode:

```csharp
interface IFeatureTableSelector
{
    Task<IReadOnlyList<RelatedFeatureQueryResult>> GetRelatedRecords(
                                                        Feature feature,
                                                        RelationshipInfo relationshipInfo);
    IReadOnlyList<RelationshipInfo> GetRelationshipInfos(Feature feature);
}
```

Create 2 classes, one for the Online table mode and one for the Offline table mode that implement your new interface:

### Online

```csharp
class OnlineFeatureTable : IFeatureTableSelector
{
    public async Task<IReadOnlyList<RelatedFeatureQueryResult>> GetRelatedRecords(Feature feature, RelationshipInfo relationshipInfo)
    {
        var parameters = new RelatedQueryParameters(relationshipInfo);
        var table = feature.FeatureTable as ServiceFeatureTable;
        var relationships = await (table).QueryRelatedFeaturesAsync(
                                                (ArcGISFeature)feature,
                                                parameters,
                                                QueryFeatureFields.LoadAll);
        return relationships;
    }

    public IReadOnlyList<RelationshipInfo> GetRelationshipInfos(Feature feature)
    {
        return ((ServiceFeatureTable)feature.FeatureTable).LayerInfo.RelationshipInfos;
    }
}
```

### Offline

```csharp
class OfflineFeatureTable : IFeatureTableSelector
{
    public async Task<IReadOnlyList<RelatedFeatureQueryResult>> GetRelatedRecords(Feature feature, RelationshipInfo relationshipInfo)
    {
        var parameters = new RelatedQueryParameters(relationshipInfo);
        var table = feature.FeatureTable as GeodatabaseFeatureTable;
        var relationships = await (table).QueryRelatedFeaturesAsync(
                                                (ArcGISFeature)feature,
                                                parameters);
        return relationships;
    }

    public IReadOnlyList<RelationshipInfo> GetRelationshipInfos(Feature feature)
    {
        return ((GeodatabaseFeatureTable)feature.FeatureTable).LayerInfo.RelationshipInfos;
    }
}
```

Once you have it all set up, you'll just determine the type of table one time, as you start working with your data, and off you go.

```csharp
IFeatureTableSelector tableSelector = (AppState == AppState.Online) ?
    new OnlineFeatureTable() as IFeatureTableSelector :
    new OfflineFeatureTable() as IFeatureTableSelector;

// get RelationshipInfos from online or offline table
var relationshipInfos = tableSelector.GetRelationshipInfos(feature);

// get Relationships from online or offline table
var relationships = await tableSelector.GetRelatedRecords(feature, relationshipInfo);
```
