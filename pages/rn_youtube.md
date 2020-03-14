# A better way to integrate youtube into react native

Videos are a great way to communicate with your users. In a customer facing ecosystem, youtube plays a part in growth, brand value and engagement. Uploading a video to youtube is easy, cost effective, easy to advertize and might actually make you money in the long run. Youtube is a household name and users are comfortable clicking on a youtube video.

When you're building a mobile app, it just makes sense to include youtube. You can put in all sorts of videos in your app like -

- promotional videos
- tutorials
- podcasts / periodic content updates

## React Native

The most popular way of embedding a youtube video in react native is to use `react-native-youtube`.

### The Problem

When you're writing apps in react native, one of the most important requirements is to make it look consistent across android and iOS. This will be a problem when you want to embed a youtube video in your app using this library.

It uses the native `YouTube Android Player API` for android and `YoutubePlayer-in-WKWebView` in iOS. The iOS version is the youtube web player and behaves very different to the android implementation. The android player API is a horrible thing to work with. It has a ton of bugs that causes your app to unexpectedly crash.

### The Solution

Using the web player on both android and iOS versions. The [react-native-youtube-iframe](https://github.com/LonelyCpp/react-native-youtube-iframe) library does exactly this.

Its a wrapper around the youtube iframe API used on the web. It uses the community maintained `react-native-webview` to drive everything.

#### Installing

1. `npm install react-native-webview` (you might already have this installed)
2. `npm install react-native-youtube-iframe`
3. `pod install` (in the iOS directory)

Thats it!

#### Code

Lets make an app that has a youtube video on top which the user can play -

```javascript
import React from "react";
import { SafeAreaView } from "react-native";
import YoutubePlayer from "react-native-youtube-iframe";

const App = () => {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <YoutubePlayer height={250} videoId={"sNhhvQGsMEc"} />
    </SafeAreaView>
  );
};

export default App;
```

Thats it! It really is that simple and it looks exactly the same on both platforms!

Multiple `<YoutubePlayer />` components with different `videoId`s can be in the same page too!

![screenshot](../assets/rn_yt_1.png?raw=true)

### MORE

Lets build a more complicated app with some of the features provided by the library.

The app features -

- a list of videos
- persist where the user last left the video
- track completion progress

#### A List of Videos

We can use a `flatlist` to render a list which will have the video thumbnail, and title of the video.

Create a list of videos you'd like to show -

```javascript
const videoSeries = [
  "DC471a9qrU4",
  "tVCYa_bnITg",
  "K74l26pE4YA",
  "m3OjWNFREJo"
];
```

```JSX
<FlatList
        contentContainerStyle={{margin: 16}}
        ListHeaderComponent={
          <>
            <Text style={{fontSize: 18, fontWeight: 'bold'}}>
              100 Seconds of Code
            </Text>
          </>
        }
        data={videoSeries}
        renderItem={({item}) => (
          <VideoItem videoId={item} onPress={onVideoPress} />
        )}
        keyExtractor={item => item}
      />
```

To get the thumbnail, title and author use the `getYoutubeMeta` function provided by the package.

```JSX
const VideoItem = ({videoId, onPress}) => {
  const [videoMeta, setVideoMeta] = useState(null);
  useEffect(() => {
    getYoutubeMeta(videoId).then(data => {
      setVideoMeta(data);
    });
  }, [videoId]);

  if (videoMeta) {
    return (
      <TouchableOpacity
        onPress={() => onPress(videoId)}
        style={{flexDirection: 'row', marginVertical: 16}}>
        <Image
          source={{uri: videoMeta.thumbnail_url}}
          style={{
            width: videoMeta.thumbnail_width / 4,
            height: videoMeta.thumbnail_height / 4,
          }}
        />
        <View style={{justifyContent: 'center', marginStart: 16}}>
          <Text style={{marginVertical: 4, fontWeight: 'bold'}}>
            {videoMeta.title}
          </Text>
          <Text>{videoMeta.author_name}</Text>
        </View>
      </TouchableOpacity>
    );
  }
  return null;
};
```

![screenshot](../assets/rn_yt_2.png?raw=true)
