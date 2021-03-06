## Data Collection for .NET

Data is at the heart of every business. And collecting it is being done in a variety of ways across industries, from paper to custom built applications. You have asked us to release a collection app that you can configure and customize to work with your own data and specific workflows. We're proud to deliver something that hopefully comes close to what you need. And because we're releasing the app open source and under an Apache license, you are welcome to take it, modify it to match your needs and use it as you see fit. And if you feel that some of the changes you have made could be beneficial to someone else in the community, we kindly ask that you contribute back to the source repository.

Data Collection for .NET is a WPF application built using the ArcGIS .NET Runtime API and it features common functionality encountered when collecting data.

### Identify map data

![identify](https://user-images.githubusercontent.com/20545379/51622849-547b6980-1eec-11e9-8e7f-6495cd4d1ca5.gif)

Collecting data means also knowing what is around you. Click on one of the data points on the map to bring up information about it. And if you need to edit it, clicking on the pencil icon will open it in edit mode. Simple as that!

### Collect new data point

![add_record](https://user-images.githubusercontent.com/20545379/51622891-652bdf80-1eec-11e9-9380-32ab30e2a88f.gif)

When you're ready to add a new data point, click the plus button to begin. You'll be prompted to select a location for your new data point and add some information about it.

### Offline Mode

![offline](https://user-images.githubusercontent.com/20545379/51622920-7379fb80-1eec-11e9-9b0a-ff191240749b.gif)

Because we know what a lot of our users work in remote areas, we have added an offline mode to the application. Simply navigate the map to your desired work area and select "Work Offline" from the menu. The app will download necessary data and will allow you to collect data points when you are disconnected from the network. And when you're back in the office or able to connect to a network, simply select "Sync Map" to have your changes merged with the online version of your map.

Do you think this is something you could use? Take a look at the [full documentation](https://developers.arcgis.com/example-apps/data-collection-dotnet/) and download the app from [GitHub](https://github.com/Esri/data-collection-dotnet)!

Happy collecting!