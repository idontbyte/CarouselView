# CarouselView control for Xamarin Forms

#### Setup
* Available on NuGet: https://www.nuget.org/packages/CarouselView.FormsPlugin/ [![NuGet](https://img.shields.io/nuget/v/CarouselView.FormsPlugin.svg?label=NuGet)](https://www.nuget.org/packages/CarouselView.FormsPlugin/)
* Install in your PCL project and Client projects.

**Platform Support**

|Platform|Supported|Version|Renderer|
| ------------------- | :-----------: | :-----------: | :------------------: |
|Xamarin.iOS Unified|Yes|iOS 8.1+|UIPageViewController|
|Xamarin.Android|Yes|API 15+|ViewPager|
|UWP|Yes|10+|FlipView|

#### Usage

In your iOS and Android projects call:

```
Xamarin.Forms.Init();
CarouselViewRenderer.Init();
```

**C#:**

```
var myCarousel = new CarouselViewControl();
myCarousel.ItemsSource = new List<int> { 1, 2, 3, 4, 5 };
myCarousel.ItemTemplate = new MyTemplateSelector (); //new DataTemplate (typeof(MyView));
myCarousel.Position = 0; //default
myCarousel.InterPageSpacing = 10;
myCarousel.Orientation = Orientation.Horizontal;
```

**XAML:**

First add the xmlns namespace:

```xml
xmlns:controls="clr-namespace:CarouselView.FormsPlugin.Abstractions;assembly=CarouselView.FormsPlugin.Abstractions"
```

Then add the xaml:

```xml
<controls:CarouselViewControl Orientation="Horizontal" InterPageSpacing="10" Position="0" ItemsSource="{Binding Pages}" VerticalOptions="FillAndExpand" HorizontalOptions="FillAndExpand">
    <controls:CarouselViewControl.ItemTemplate>
        <DataTemplate>
            <local:MyView />
	    </DataTemplate>
    </controls:CarouselViewControl.ItemTemplate>
</controls:CarouselViewControl>
```

Or, you can use a data template selector.

```xml
<ContentPage.Resources>
	 <ResourceDictionary>
	   <local:MyTemplateSelector x:Key="myTemplateSelector"></local:MyTemplateSelector>
	 </ResourceDictionary>
</ContentPage.Resources>
```

Then the xaml:

```xml
<controls:CarouselViewControl Position="0" ItemsSource="{Binding Pages}" ItemTemplate="{StaticResource myTemplateSelector}" VerticalOptions="FillAndExpand" HorizontalOptions="FillAndExpand"/>
```

**HEIGHT REQUEST**

As a requirement you have to provide HeighRequest for UI components using OnPlatform:

```xml
<Button Text="MyButton">
    <Button.HeightRequest>
        <OnPlatform x:TypeArguments="x:Double"
            Android="48"
            WinPhone="32"
            iOS="44" />
    </Button.HeightRequest>
</Button>
```

```
Device.OnPlatform(
    iOS: () => myButton.HeightRequest = 44,
    Android: () => myButton.HeightRequest = 48,
    WinPhone: ()=> myButton.HeightRequest = 32
);
```

**Default HeightRequest by platform**

|Control|iOS|Android|UWP|
| ------------------- | :-----------: | :-----------: | :------------------: |
|ActivityIndicator|20|48|4|
|Button|44|48|32|
|DatePicker|30|45.5|32|
|Editor|36.5|45.5|32|
|Entry|30|45.5|32|
|Picker|30|45.5|32|
|ProgressBar|2|16|4|
|Searchbar|44|45|32|
|Slider|34|18|44|
|Stepper|29|48|32|
|Switch|31|27|40|
|TimePicker|30|45.5|32|

#### I provide CVLabel to solve HeightRequest issue. If you have your own Label renderer you need to add this code snippet:

```
public class CVLabel : Label
{
	public CVLabel()
	{
		SetBinding(Label.HeightRequestProperty, new Binding("WidthRequest", BindingMode.Default, new LabelHeightConverter(), this, null, this));
	}

	protected override void OnSizeAllocated(double width, double height)
	{
		base.OnSizeAllocated(width, height);

		WidthRequest = width;

		this.LayoutTo(new Rectangle(this.X, this.Y, width, height));

		this.InvalidateMeasure();
	}
}
```

**Bindable Properties**

```Orientation```: Vertical or Horizontal swipe/scroll (default horizontal).

```ItemsSource```: Collection of objects used as BindingContext of each view.

* You can use an Observable collection as ItemsSource and use CollectionChanged events in your end for your own business logic.

```ItemTemplate```: supports DataTemplate and DataTemplateSelector.

```Position```: the desired selected view when Carousel starts.

```Bounces```: use this property to disable bounces when you will render one page at a time and move back and fort programmatically (iOS only, default true).

```Arrows```: disable arrows navigation (UWP only, default true).

```PageIndicators```: hide/show page indicators (default false).

```PageIndicatorTintColor```: page dot indicators fill color (default #C0C0C0).

```CurrentPageIndicatorTintColor```: selected page dot indicator fill color (default #808080).

```InterPageSpacing```: add a margin/space between pages (Android and iOS only).

**Event Handlers**

```PositionSelected```: called when position changes.

**Methods**

```RemovePage (position)```: remove a view at given position (when you remove the current view it will slide to the previous one). This method will also remove the related item from the ItemsSource.

```InsertPage (item, position)```: insert a view at a given position (if position parameter is not provided, item will be added at the end).

```SetCurrentPage (position)```: slide programmatically to a given position.

#### Render one page at a time, no swiping, move back and fort programmatically:

```
var pages = new List<int> { 1 }; // only one item in ItemsSource
Carousel.ItemsSource = pages;
```

To move forward:

```
void OnNext (object sender, TappedEventArgs e) {
	Carousel.InsertPage(2); // parameter is the new item to be used as binding context for the second view
	Carousel.RemovePage(0);
}
```

To move back:

```
async void OnPrevious (object sender, TappedEventArgs e) {
	var pages = new List<int>() { 1, pages[0] }; // inserting one item on position 0 of ItemsSource
	Carousel.Position = 1;
	Carousel.ItemsSource = pages;
	await Task.Delay(100);
	Carousel.RemovePage(1);
}
```

#### Known issues

- Horizontal StackLayout doesn't works. Why? No idea :) You may use a multi-column Grid instead.
- ListView inside a page of CarouselView doesn't works with Xamarin.Forms versions bigger than 2.3.0.107 as there's a bug with Platform.SetRenderer API (by the way latest Xamarin.Forms releases are buggy).

#### Tips

- If you have memory leaks in Android when using the Carousel with images, it's not the control itself. It's Xamarin.Forms Android not handling images correctly. To solve the problem you can use [FFImageLoading](https://github.com/luberda-molinet/FFImageLoading) making sure that you set this properties:

```
DownsampleToViewSize="true" DownsampleWidth="WIDTH"
```

#### Contributors
* [alexrainman](https://github.com/alexrainman)

Thanks!

#### License
Licensed under repo license
