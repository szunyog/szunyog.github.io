---
layout: post
title:  "WPF TabControl First, Last TabItem Style"
date:   2015-02-10 20:31:49
categories: csharp
tags: [wpf, tabcontrol]
---

Using HTML and CSS is quite easy to apply a custom style to the first or last elements of a container using the ``:first-child`` or ``:last-child`` pseudo elements. But in WPF it is a bit harder, because it does not support anything like that.

But using a custom value converter it is possible to specify specific markers for the first or last elements of a tabcontrol.

{% highlight csharp %}

public class TabIndexConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        TabItem tabItem = value as TabItem;
        var fromContainer = ItemsControl.ItemsControlFromItemContainer(tabItem).ItemContainerGenerator;

        var items = fromContainer.Items.Cast<TabItem>().Where(x => x.Visibility == Visibility.Visible).ToList();
        var count = items.Count();

        var index = items.IndexOf(tabItem);
        if (index == 0)
            return "First";
        else if (count - 1 == index)
            return "Last";
        else
            return "";
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return DependencyProperty.UnsetValue;
    }
}

{% endhighlight %}

And using the provided values of the converter, a "simple" style can be defined to apply custom appearance to the first or last tab items. In this example only the first and last item is rounded.

{% highlight xml %}

...
    <c:TabIndexConverter x:Key="TabIndexConverter" />
    <SolidColorBrush x:Key="TabItemSelectedBackground" Color="Gray" />
    <SolidColorBrush x:Key="StandardButtonBackground" Color="White" />

    <Style TargetType="{x:Type TabItem}">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="TabItem">
                    <Grid Name="Panel" Height="90">
                        <Border Name="Border" Background="White" BorderBrush="{StaticResource ResourceKey=TabItemSelectedBackground}" BorderThickness="1,1,1,1" CornerRadius="0,0,0,0" Margin="0,10,0,10">
                            <ContentPresenter x:Name="ContentSite" VerticalAlignment="Center" HorizontalAlignment="Center" ContentSource="Header" Margin="30,10,30,10"/>
                        </Border>
                    </Grid>
                    <ControlTemplate.Triggers>
                        <Trigger Property="IsSelected" Value="True">
                            <Setter TargetName="Border" Property="Background" Value="{StaticResource ResourceKey=TabItemSelectedBackground}" />
                        </Trigger>
                        <Trigger Property="IsSelected" Value="False">
                            <Setter TargetName="Border" Property="Background" Value="{StaticResource ResourceKey=StandardButtonBackground}" />
                        </Trigger>
                        <DataTrigger Binding="{Binding Converter={StaticResource TabIndexConverter}, RelativeSource={RelativeSource Self}}" Value="First">
                            <Setter TargetName="Border" Property="CornerRadius" Value="40,0,0,40" />
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Converter={StaticResource TabIndexConverter}, RelativeSource={RelativeSource Self}}" Value="Last">
                            <Setter TargetName="Border" Property="CornerRadius" Value="0,40,40,0" />
                        </DataTrigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
    ...

{% endhighlight %}


