# Documentation for `docs/samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml_docs.md`
- **File Name**: `StandardStyles.xaml_docs.md`
- **File Size**: 50,817 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml_docs.md](../../../../../../docs/samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/OcvImageProcessing/OcvImageProcessing/Common` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml`

## File Metadata

- **Full Path**: `samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml`
- **File Name**: `StandardStyles.xaml`
- **File Size**: 117,191 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml](../../../../../samples/winrt/OcvImageProcessing/OcvImageProcessing/Common/StandardStyles.xaml)

## Purpose and Role

This file is located in the `samples/winrt/OcvImageProcessing/OcvImageProcessing/Common` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<!--
    This file contains XAML styles that simplify application development.

    These are not merely convenient, but are required by most Visual Studio project and item templates.
    Removing, renaming, or otherwise modifying the content of these files may result in a project that
    does not build, or that will not build once additional pages are added.  If variations on these
    styles are desired it is recommended that you copy the content under a new name and modify your
    private copy.
-->

<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <!-- Non-brush values that vary across themes -->

    <ResourceDictionary.ThemeDictionaries>
        <ResourceDictionary x:Key="Default">
            <x:String x:Key="BackButtonGlyph">&#xE071;</x:String>
            <x:String x:Key="BackButtonSnappedGlyph">&#xE0BA;</x:String>
        </ResourceDictionary>

        <ResourceDictionary x:Key="HighContrast">
            <x:String x:Key="BackButtonGlyph">&#xE071;</x:String>
            <x:String x:Key="BackButtonSnappedGlyph">&#xE0C4;</x:String>
        </ResourceDictionary>
    </ResourceDictionary.ThemeDictionaries>

    <x:String x:Key="ChevronGlyph">&#xE26B;</x:String>

    <!-- RichTextBlock styles -->

    <Style x:Key="BasicRichTextStyle" TargetType="RichTextBlock">
        <Setter Property="Foreground" Value="{StaticResource ApplicationForegroundThemeBrush}"/>
        <Setter Property="FontSize" Value="{StaticResource ControlContentThemeFontSize}"/>
        <Setter Property="FontFamily" Value="{StaticResource ContentControlThemeFontFamily}"/>
        <Setter Property="TextTrimming" Value="WordEllipsis"/>
        <Setter Property="TextWrapping" Value="Wrap"/>
        <Setter Property="Typography.StylisticSet20" Value="True"/>
        <Setter Property="Typography.DiscretionaryLigatures" Value="True"/>
        <Setter Property="Typography.CaseSensitiveForms" Value="True"/>
    </Style>

    <Style x:Key="BaselineRichTextStyle" TargetType="RichTextBlock" BasedOn="{StaticResource BasicRichTextStyle}">
        <Setter Property="LineHeight" Value="20"/>
        <Setter Property="LineStackingStrategy" Value="BlockLineHeight"/>
        <!-- Properly align text along its baseline -->
        <Setter Property="RenderTransform">
            <Setter.Value>
                <TranslateTransform X="-1" Y="4"/>
            </Setter.Value>
        </Setter>
    </Style>

    <Style x:Key="ItemRichTextStyle" TargetType="RichTextBlock" BasedOn="{StaticResource BaselineRichTextStyle}"/>

    <Style x:Key="BodyRichTextStyle" TargetType="RichTextBlock" BasedOn="{StaticResource BaselineRichTextStyle}">
        <Setter Property="FontWeight" Value="SemiLight"/>
    </Style>

    <!-- TextBlock styles -->

    <Style x:Key="BasicTextStyle" TargetType="TextBlock">
        <Setter Property="Foreground" Value="{StaticResource ApplicationForegroundThemeBrush}"/>
        <Setter Property="FontSize" Value="{StaticResource ControlContentThemeFontSize}"/>
        <Setter Property="FontFamily" Value="{StaticResource ContentControlThemeFontFamily}"/>
        <Setter Property="TextTrimming" Value="WordEllipsis"/>
        <Setter Property="TextWrapping" Value="Wrap"/>
        <Setter Property="Typography.StylisticSet20" Value="True"/>
        <Setter Property="Typography.DiscretionaryLigatures" Value="True"/>
        <Setter Property="Typography.CaseSensitiveForms" Value="True"/>
    </Style>

    <Style x:Key="BaselineTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BasicTextStyle}">
        <Setter Property="LineHeight" Value="20"/>
        <Setter Property="LineStackingStrategy" Value="BlockLineHeight"/>
        <!-- Properly align text along its baseline -->
        <Setter Property="RenderTransform">
            <Setter.Value>
                <TranslateTransform X="-1" Y="4"/>
            </Setter.Value>
        </Setter>
    </Style>

    <Style x:Key="HeaderTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontSize" Value="56"/>
        <Setter Property="FontWeight" Value="Light"/>
        <Setter Property="LineHeight" Value="40"/>
        <Setter Property="RenderTransform">
            <Setter.Value>
                <TranslateTransform X="-2" Y="8"/>
            </Setter.Value>
        </Setter>
    </Style>

    <Style x:Key="SubheaderTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontSize" Value="26.667"/>
        <Setter Property="FontWeight" Value="Light"/>
        <Setter Property="LineHeight" Value="30"/>
        <Setter Property="RenderTransform">
            <Setter.Value>
                <TranslateTransform X="-1" Y="6"/>
            </Setter.Value>
        </Setter>
    </Style>

    <Style x:Key="TitleTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontWeight" Value="SemiBold"/>
    </Style>

    <Style x:Key="SubtitleTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontWeight" Value="Normal"/>
    </Style>

    <Style x:Key="ItemTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}"/>

    <Style x:Key="BodyTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontWeight" Value="SemiLight"/>
    </Style>

    <Style x:Key="CaptionTextStyle" TargetType="TextBlock" BasedOn="{StaticResource BaselineTextStyle}">
        <Setter Property="FontSize" Value="12"/>
        <Setter Property="Foreground" Value="{StaticResource ApplicationSecondaryForegroundThemeBrush}"/>
    </Style>

    <Style x:Key="GroupHeaderTextStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="{StaticResource ContentControlThemeFontFamily}"/>
        <Setter Property="TextTrimming" Value="WordEllipsis"/>
        <Setter Property="TextWrapping" Value="NoWrap"/>
        <Setter Property="Typography.StylisticSet20" Value="True"/>
        <Setter Property="Typography.DiscretionaryLigatures" Value="True"/>
        <Setter Property="Typography.CaseSensitiveForms" Value="True"/>
        <Setter Property="FontSize" Value="26.667"/>
        <Setter Property="LineStackingStrategy" Value="BlockLineHeight"/>
        <Setter Property="FontWeight" Value="Light"/>
        <Setter Property="LineHeight" Value="30"/>
        <Setter Property="RenderTransform">
            <Setter.Value>
                <TranslateTransform X="-1" Y="6"/>
            </Setter.Value>
        </Setter>
    </Style>

    <!-- Button styles -->
 <!--
        TextButtonStyle is used to style a Button using subheader-styled text with no other adornment.  There
        are two styles that are based on TextButtonStyle (TextPrimaryButtonStyle and TextSecondaryButtonStyle)
        which are used in the GroupedItemsPage as a group header and in the FileOpenPickerPage for triggering
        commands.
    -->
    <Style x:Key="TextButtonStyle" TargetType="ButtonBase">
        <Setter Property="MinWidth" Value="0"/>
        <Setter Property="MinHeight" Value="0"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="ButtonBase">
                    <Grid Background="Transparent">
                        <ContentPresenter x:Name="Text" Content="{TemplateBinding Content}" />
                        <Rectangle
                            x:Name="FocusVisualWhite"
                            IsHitTestVisible="False"
                            Stroke="{StaticResource FocusVisualWhiteStrokeThemeBrush}"
                            StrokeEndLineCap="Square"
                            StrokeDashArray="1,1"
                            Opacity="0"
                            StrokeDashOffset="1.5"/>
                        <Rectangle
                            x:Name="FocusVisualBlack"
                            IsHitTestVisible="False"
                            Stroke="{StaticResource FocusVisualBlackStrokeThemeBrush}"
                            StrokeEndLineCap="Square"
                            StrokeDashArray="1,1"
                            Opacity="0"
                            StrokeDashOffset="0.5"/>
                        <VisualStateManager.VisualStateGroups>
                            <VisualStateGroup x:Name="CommonStates">
                                <VisualState x:Name="Normal"/>
                                <VisualState x:Name="PointerOver">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Text" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource ApplicationPointerOverForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Pressed">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Text" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource ApplicationPressedForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Disabled">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Text" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource ApplicationPressedForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                            </VisualStateGroup>
                            <VisualStateGroup x:Name="FocusStates">
                                <VisualState x:Name="Focused">
                                    <Storyboard>
                                        <DoubleAnimation Duration="0" To="1" Storyboard.TargetName="FocusVisualWhite" Storyboard.TargetProperty="Opacity"/>
                                        <DoubleAnimation Duration="0" To="1" Storyboard.TargetName="FocusVisualBlack" Storyboard.TargetProperty="Opacity"/>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Unfocused"/>
                            </VisualStateGroup>
                            <VisualStateGroup x:Name="CheckStates">
                                <VisualState x:Name="Checked"/>
                                <VisualState x:Name="Unchecked">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Text" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource ApplicationSecondaryForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Indeterminate"/>
                            </VisualStateGroup>
                        </VisualStateManager.VisualStateGroups>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <Style x:Key="TextPrimaryButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource TextButtonStyle}">
        <Setter Property="Foreground" Value="{StaticResource ApplicationHeaderForegroundThemeBrush}"/>
    </Style>

    <Style x:Key="TextSecondaryButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource TextButtonStyle}">
        <Setter Property="Foreground" Value="{StaticResource ApplicationSecondaryForegroundThemeBrush}"/>
    </Style>

    <!--
        TextRadioButtonStyle is used to style a RadioButton using subheader-styled text with no other adornment.
        This style is used in the SearchResultsPage to allow selection among filters.
    -->
    <Style x:Key="TextRadioButtonStyle" TargetType="RadioButton" BasedOn="{StaticResource TextButtonStyle}">
        <Setter Property="Margin" Value="0,0,30,0"/>
    </Style>

    <!--
        AppBarButtonStyle is used to style a Button (or ToggleButton) for use in an App Bar.  Content will be centered
        and should fit within the 40 pixel radius glyph provided.  16-point Segoe UI Symbol is used for content text
        to simplify the use of glyphs from that font.  AutomationProperties.Name is used for the text below the glyph.
    -->
    <Style x:Key="AppBarButtonStyle" TargetType="ButtonBase">
        <Setter Property="Foreground" Value="{StaticResource AppBarItemForegroundThemeBrush}"/>
        <Setter Property="VerticalAlignment" Value="Stretch"/>
        <Setter Property="FontFamily" Value="Segoe UI Symbol"/>
        <Setter Property="FontWeight" Value="Normal"/>
        <Setter Property="FontSize" Value="20"/>
        <Setter Property="AutomationProperties.ItemType" Value="App Bar Button"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="ButtonBase">
                    <Grid x:Name="RootGrid" Width="100" Background="Transparent">
                        <StackPanel VerticalAlignment="Top" Margin="0,12,0,11">
                            <Grid Width="40" Height="40" Margin="0,0,0,5" HorizontalAlignment="Center">
                                <TextBlock x:Name="BackgroundGlyph" Text="&#xE0A8;" FontFamily="Segoe UI Symbol" FontSize="53.333" Margin="-4,-19,0,0" Foreground="{StaticResource AppBarItemBackgroundThemeBrush}"/>
                                <TextBlock x:Name="OutlineGlyph" Text="&#xE0A7;" FontFamily="Segoe UI Symbol" FontSize="53.333" Margin="-4,-19,0,0"/>
                                <ContentPresenter x:Name="Content" HorizontalAlignment="Center" Margin="-1,-1,0,0" VerticalAlignment="Center"/>
                            </Grid>
                            <TextBlock
                                x:Name="TextLabel"
                                Text="{TemplateBinding AutomationProperties.Name}"
                                Foreground="{StaticResource AppBarItemForegroundThemeBrush}"
                                Margin="0,0,2,0"
                                FontSize="12"
                                TextAlignment="Center"
                                Width="88"
                                MaxHeight="32"
                                TextTrimming="WordEllipsis"
                                Style="{StaticResource BasicTextStyle}"/>
                        </StackPanel>
                        <Rectangle
                                x:Name="FocusVisualWhite"
                                IsHitTestVisible="False"
                                Stroke="{StaticResource FocusVisualWhiteStrokeThemeBrush}"
                                StrokeEndLineCap="Square"
                                StrokeDashArray="1,1"
                                Opacity="0"
                                StrokeDashOffset="1.5"/>
                        <Rectangle
                                x:Name="FocusVisualBlack"
                                IsHitTestVisible="False"
                                Stroke="{StaticResource FocusVisualBlackStrokeThemeBrush}"
                                StrokeEndLineCap="Square"
                                StrokeDashArray="1,1"
                                Opacity="0"
                                StrokeDashOffset="0.5"/>

                        <VisualStateManager.VisualStateGroups>
                            <VisualStateGroup x:Name="ApplicationViewStates">
                                <VisualState x:Name="FullScreenLandscape"/>
                                <VisualState x:Name="Filled"/>
                                <VisualState x:Name="FullScreenPortrait">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TextLabel" Storyboard.TargetProperty="Visibility">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="Collapsed"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="RootGrid" Storyboard.TargetProperty="Width">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="60"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Snapped">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TextLabel" Storyboard.TargetProperty="Visibility">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="Collapsed"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="RootGrid" Storyboard.TargetProperty="Width">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="60"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                            </VisualStateGroup>
                            <VisualStateGroup x:Name="CommonStates">
                                <VisualState x:Name="Normal"/>
                                <VisualState x:Name="PointerOver">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="BackgroundGlyph" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemPointerOverBackgroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Content" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemPointerOverForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Pressed">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="OutlineGlyph" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="BackgroundGlyph" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Content" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemPressedForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Disabled">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="OutlineGlyph" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemDisabledForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Content" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemDisabledForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TextLabel" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemDisabledForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                            </VisualStateGroup>
                            <VisualStateGroup x:Name="FocusStates">
                                <VisualState x:Name="Focused">
                                    <Storyboard>
                                        <DoubleAnimation
                                                Storyboard.TargetName="FocusVisualWhite"
                                                Storyboard.TargetProperty="Opacity"
                                                To="1"
                                                Duration="0"/>
                                        <DoubleAnimation
                                                Storyboard.TargetName="FocusVisualBlack"
                                                Storyboard.TargetProperty="Opacity"
                                                To="1"
                                                Duration="0"/>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Unfocused" />
                                <VisualState x:Name="PointerFocused" />
                            </VisualStateGroup>
                            <VisualStateGroup x:Name="CheckStates">
                                <VisualState x:Name="Checked">
                                    <Storyboard>
                                        <DoubleAnimation Duration="0" To="0" Storyboard.TargetName="OutlineGlyph" Storyboard.TargetProperty="Opacity"/>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="BackgroundGlyph" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="BackgroundCheckedGlyph" Storyboard.TargetProperty="Visibility">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="Visible"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Content" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{StaticResource AppBarItemPressedForegroundThemeBrush}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Unchecked"/>
                                <VisualState x:Name="Indeterminate"/>
                            </VisualStateGroup>
                        </VisualStateManager.VisualStateGroups>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <!--
        Standard AppBarButton Styles for use with Button and ToggleButton

  An AppBarButton Style is provided for each of the glyphs in the Segoe UI Symbol font.
       Uncomment any style you reference (as not all may be required).
    -->

    <!--
 <
Style x:Key="SkipBackAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SkipBackAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Skip Back"/>
        <Setter Property="Content" Value="&#xE100;"/>
    </Style>
    <Style x:Key="SkipAheadAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SkipAheadAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Skip Ahead"/>
        <Setter Property="Content" Value="&#xE101;"/>
    </Style>
    <Style x:Key="PlayAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PlayAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Play"/>
        <Setter Property="Content" Value="&#xE102;"/>
    </Style>
    <Style x:Key="PauseAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PauseAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Pause"/>
        <Setter Property="Content" Value="&#xE103;"/>
    </Style>
    <Style x:Key="EditAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="EditAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Edit"/>
        <Setter Property="Content" Value="&#xE104;"/>
    </Style>
    <Style x:Key="SaveAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SaveAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Save"/>
        <Setter Property="Content" Value="&#xE105;"/>
    </Style>
    <Style x:Key="DeleteAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DeleteAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Delete"/>
        <Setter Property="Content" Value="&#xE106;"/>
    </Style>
    <Style x:Key="DiscardAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DiscardAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Discard"/>
        <Setter Property="Content" Value="&#xE107;"/>
    </Style>
    <Style x:Key="RemoveAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RemoveAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Remove"/>
        <Setter Property="Content" Value="&#xE108;"/>
    </Style>
    <Style x:Key="AddAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="AddAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Add"/>
        <Setter Property="Content" Value="&#xE109;"/>
    </Style>
    <Style x:Key="NoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="NoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="No"/>
        <Setter Property="Content" Value="&#xE10A;"/>
    </Style>
    <Style x:Key="YesAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="YesAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Yes"/>
        <Setter Property="Content" Value="&#xE10B;"/>
    </Style>
    <Style x:Key="MoreAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="MoreAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="More"/>
        <Setter Property="Content" Value="&#xE10C;"/>
    </Style>
    <Style x:Key="RedoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RedoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Redo"/>
        <Setter Property="Content" Value="&#xE10D;"/>
    </Style>
    <Style x:Key="UndoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="UndoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Undo"/>
        <Setter Property="Content" Value="&#xE10E;"/>
    </Style>
    <Style x:Key="HomeAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="HomeAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Home"/>
        <Setter Property="Content" Value="&#xE10F;"/>
    </Style>
    <Style x:Key="OutAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="OutAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Out"/>
        <Setter Property="Content" Value="&#xE110;"/>
    </Style>
    <Style x:Key="NextAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="NextAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Next"/>
        <Setter Property="Content" Value="&#xE111;"/>
    </Style>
    <Style x:Key="PreviousAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PreviousAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Previous"/>
        <Setter Property="Content" Value="&#xE112;"/>
    </Style>
    <Style x:Key="FavoriteAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="FavoriteAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Favorite"/>
        <Setter Property="Content" Value="&#xE113;"/>
    </Style>
    <Style x:Key="PhotoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PhotoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Photo"/>
        <Setter Property="Content" Value="&#xE114;"/>
    </Style>
    <Style x:Key="SettingsAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SettingsAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Settings"/>
        <Setter Property="Content" Value="&#xE115;"/>
    </Style>
    -->

    <!--
    <Style x:Key="VideoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="VideoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Video"/>
        <Setter Property="Content" Value="&#xE116;"/>
    </Style>
    <Style x:Key="RefreshAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RefreshAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Refresh"/>
        <Setter Property="Content" Value="&#xE117;"/>
    </Style>
    <Style x:Key="DownloadAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DownloadAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Download"/>
        <Setter Property="Content" Value="&#xE118;"/>
    </Style>
    <Style x:Key="MailAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="MailAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Mail"/>
        <Setter Property="Content" Value="&#xE119;"/>
    </Style>
    <Style x:Key="SearchAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SearchAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Search"/>
        <Setter Property="Content" Value="&#xE11A;"/>
    </Style>
    <Style x:Key="HelpAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="HelpAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Help"/>
        <Setter Property="Content" Value="&#xE11B;"/>
    </Style>
    <Style x:Key="UploadAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="UploadAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Upload"/>
        <Setter Property="Content" Value="&#xE11C;"/>
    </Style>
    <Style x:Key="EmojiAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="EmojiAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Emoji"/>
        <Setter Property="Content" Value="&#xE11D;"/>
    </Style>
    <Style x:Key="TwoPageAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="TwoPageAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Two Page"/>
        <Setter Property="Content" Value="&#xE11E;"/>
    </Style>
    <Style x:Key="LeaveChatAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="LeaveChatAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Upload"/>
        <Setter Property="Content" Value="&#xE11F;"/>
    </Style>
    <Style x:Key="MailForwardAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="MailForwardAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Forward Mail"/>
        <Setter Property="Content" Value="&#xE120;"/>
    </Style>
    <Style x:Key="ClockAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ClockAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Clock"/>
        <Setter Property="Content" Value="&#xE121;"/>
    </Style>
    <Style x:Key="SendAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SendAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Send"/>
        <Setter Property="Content" Value="&#xE122;"/>
    </Style>
    <Style x:Key="CropAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="CropAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Crop"/>
        <Setter Property="Content" Value="&#xE123;"/>
    </Style>
    <Style x:Key="RotateCameraAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RotateCameraAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Rotate Camera"/>
        <Setter Property="Content" Value="&#xE124;"/>
    </Style>
    <Style x:Key="PeopleAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PeopleAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="People"/>
        <Setter Property="Content" Value="&#xE125;"/>
    </Style>
    <Style x:Key="ClosePaneAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ClosePaneAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Close Pane"/>
        <Setter Property="Content" Value="&#xE126;"/>
    </Style>
    <Style x:Key="OpenPaneAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="OpenPaneAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Open Pane"/>
        <Setter Property="Content" Value="&#xE127;"/>
    </Style>
    -->

    <!--
    <Style x:Key="WorldAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="WorldAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="World"/>
        <Setter Property="Content" Value="&#xE128;"/>
    </Style>
    <Style x:Key="FlagAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="FlagAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Flag"/>
        <Setter Property="Content" Value="&#xE129;"/>
    </Style>
    <Style x:Key="PreviewLinkAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PreviewLinkAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Preview Link"/>
        <Setter Property="Content" Value="&#xE12A;"/>
    </Style>
    <Style x:Key="GlobeAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="GlobeAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Globe"/>
        <Setter Property="Content" Value="&#xE12B;"/>
    </Style>
    <Style x:Key="TrimAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="TrimAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Trim"/>
        <Setter Property="Content" Value="&#xE12C;"/>
    </Style>
    <Style x:Key="AttachCameraAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="AttachCameraAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Attach Camera"/>
        <Setter Property="Content" Value="&#xE12D;"/>
    </Style>
    <Style x:Key="ZoomInAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ZoomInAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Zoom In"/>
        <Setter Property="Content" Value="&#xE12E;"/>
    </Style>
    <Style x:Key="BookmarksAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="BookmarksAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Bookmarks"/>
        <Setter Property="Content" Value="&#xE12F;"/>
    </Style>
    <Style x:Key="DocumentAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DocumentAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Document"/>
        <Setter Property="Content" Value="&#xE130;"/>
    </Style>
    <Style x:Key="ProtectedDocumentAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ProtectedDocumentAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Protected Document"/>
        <Setter Property="Content" Value="&#xE131;"/>
    </Style>
    <Style x:Key="PageAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PageAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Page"/>
        <Setter Property="Content" Value="&#xE132;"/>
    </Style>
    <Style x:Key="BulletsAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="BulletsAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Bullets"/>
        <Setter Property="Content" Value="&#xE133;"/>
    </Style>
    <Style x:Key="CommentAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="CommentAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Comment"/>
        <Setter Property="Content" Value="&#xE134;"/>
    </Style>
    <Style x:Key="Mail2AppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="Mail2AppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Mail2"/>
        <Setter Property="Content" Value="&#xE135;"/>
    </Style>
    <Style x:Key="ContactInfoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ContactInfoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Contact Info"/>
        <Setter Property="Content" Value="&#xE136;"/>
    </Style>
    <Style x:Key="HangUpAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="HangUpAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Hang Up"/>
        <Setter Property="Content" Value="&#xE137;"/>
    </Style>
    <Style x:Key="ViewAllAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ViewAllAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="View All"/>
        <Setter Property="Content" Value="&#xE138;"/>
    </Style>
    <Style x:Key="MapPinAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="MapPinAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Map Pin"/>
        <Setter Property="Content" Value="&#xE139;"/>
    </Style>
    <Style x:Key="PhoneAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PhoneAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Phone"/>
        <Setter Property="Content" Value="&#xE13A;"/>
    </Style>
    <Style x:Key="VideoChatAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="VideoChatAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Video Chat"/>
        <Setter Property="Content" Value="&#xE13B;"/>
    </Style>
    <Style x:Key="SwitchAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SwitchAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Switch"/>
        <Setter Property="Content" Value="&#xE13C;"/>
    </Style>
    <Style x:Key="ContactAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="ContactAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Contact"/>
        <Setter Property="Content" Value="&#xE13D;"/>
    </Style>

    -->

    <!--

    <Style x:Key="RenameAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RenameAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Rename"/>
        <Setter Property="Content" Value="&#xE13E;"/>
    </Style>
    <Style x:Key="PinAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="PinAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Pin"/>
        <Setter Property="Content" Value="&#xE141;"/>
    </Style>
    <Style x:Key="MusicInfoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="MusicInfoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Music Info"/>
        <Setter Property="Content" Value="&#xE142;"/>
    </Style>
    <Style x:Key="GoAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="GoAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Go"/>
        <Setter Property="Content" Value="&#xE143;"/>
    </Style>
    <Style x:Key="KeyboardAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="KeyboardAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Keyboard"/>
        <Setter Property="Content" Value="&#xE144;"/>
    </Style>
    <Style x:Key="DockLeftAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DockLeftAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Dock Left"/>
        <Setter Property="Content" Value="&#xE145;"/>
    </Style>
    <Style x:Key="DockRightAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DockRightAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Dock Right"/>
        <Setter Property="Content" Value="&#xE146;"/>
    </Style>
    <Style x:Key="DockBottomAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="DockBottomAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Dock Bottom"/>
        <Setter Property="Content" Value="&#xE147;"/>
    </Style>
    <Style x:Key="RemoteAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="RemoteAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Remote"/>
        <Setter Property="Content" Value="&#xE148;"/>
    </Style>
    <Style x:Key="SyncAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter Property="AutomationProperties.AutomationId" Value="SyncAppBarButton"/>
        <Setter Property="AutomationProperties.Name" Value="Sync"/>
        <Setter Property="Content" Value="&#xE149;"/>
    </Style>
    <Style x:Key="RotateAppBarButtonStyle" TargetType="ButtonBase" BasedOn="{StaticResource AppBarButtonStyle}">
        <Setter
... [truncated]
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

