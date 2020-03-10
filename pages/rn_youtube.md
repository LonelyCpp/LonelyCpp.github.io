# A better way to integrate youtube into react native

Videos are a great way to communicate with your users. In a customer facing ecosystem, youtube plays a part in growth, brand value and engagement. Uploading a video to youtube is easy, cost effective, easy to advertize might actually make you money in the long run. Youtube is a household name and users are comfortable clinking on a youtube video.

When you're building a mobile app, it just makes sense to include youtube. You can put in all sorts of videos in your app like -

- promotional videos
- tutorials
- podcasts / periodic content updates

## React Native

The most popular way of embedding a youtube video in react native is to use `react-native-youtube`.

### The Problem

When you're writing apps in react native, one of the most important requirements is to make it look consistent across android and iOS. This will be a problem when you want to embed a youtube video in your app using this library.

It uses the native `YouTube Android Player API` for android and `YoutubePlayer-in-WKWebView` in iOS. The iOS version is the youtube web player and behaves very different to the android implementation.

The android player API is a horrible thing to work with. It has a ton of bugs that causes your app to unexpectedly crash.

### The Solution

Using the web player on both android and iOS versions. The [`react-native-youtube-iframe`](https://github.com/LonelyCpp/react-native-youtube-iframe) library does exactly this.
