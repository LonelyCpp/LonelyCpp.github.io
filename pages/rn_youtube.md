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

{% raw  %}

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

{% endraw %}

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

(global variable)

```javascript
const videoSeries = [
  "DC471a9qrU4",
  "tVCYa_bnITg",
  "K74l26pE4YA",
  "m3OjWNFREJo"
];
```

(`App` component)

{% raw  %}

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

{% endraw  %}

To get the thumbnail, title and author use the `getYoutubeMeta` function provided by the package. As it returns a promise while it makes the API call, we will use a state variable to hold the data and render it when the promise is resolved. (A loading state could be added here)

{% raw  %}

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

{% endraw  %}

![screenshot](../assets/rn_yt_2.png?raw=true)

Now when the user clicks on the video, the app opens a modal with the youtube player on it.

Using react hooks, we can maintain states to manage the modal visibility -

(`App` component)

```javascript
const [modalVisible, showModal] = useState(false);
const [selectedVideo, setSelectedVideo] = useState(null);

const onVideoPress = useCallback(videoId => {
  showModal(true);
  setSelectedVideo(videoId);
}, []);

const closeModal = useCallback(() => showModal(false), []);
```

Modal -

(`App` component)

```JSX
<Modal
  visible={modalVisible}
  transparent={true}
  onRequestClose={closeModal}>
  <VideoModal videoId={selectedVideo} onClose={closeModal} />
</Modal>
```

```js
const VideoModal = ({ videoId, onClose }) => {
  const playerRef = useRef(null);

  return (
    <View
      style={{
        flex: 1,
        backgroundColor: "#000000dd",
        justifyContent: "center"
      }}
    >
      <View style={{ backgroundColor: "white", padding: 16 }}>
        <Text onPress={onClose} style={{ textAlign: "right" }}>
          Close
        </Text>
        <YoutubeIframe
          ref={playerRef}
          play={true}
          videoId={videoId}
          height={250}
        />
      </View>
    </View>
  );
};
```

#### Persist where the user last left the video

Install `@react-native-community/async-storage` from [here](https://github.com/react-native-community/async-storage#install)

import it into your app -
`import AsyncStorage from '@react-native-community/async-storage';`

To persist video progress, we can make a map of `videoId` to its time-stamp and completed status.

eg -

```json
{
  "DC471a9qrU4": {
    "timeStamp": 50,
    "completed": false
  }
}
```

We can use AsyncStorage to store and fetch progress in this format -

```javascript
const saveVideoProgress = ({ videoId, completed, timeStamp }) => {
  const data = {
    completed,
    timeStamp
  };

  return AsyncStorage.setItem(videoId, JSON.stringify(data));
};

const getVideoProgress = async videoId => {
  const json = await AsyncStorage.getItem(videoId);
  if (json) {
    return JSON.parse(json);
  }
  return {
    completed: false,
    timeStamp: 0
  };
};
```

Now to periodically update the timestamp, start an interval function that fetches the timestamp in regular intervals. It can be started when the `VideoModal` component mounts using `useEffect`. The `useEffect` function is called once when the component mounts to start the timer which will execute every 2 seconds here. It returns a function that clears the interval when the component is unmounted.

(`VideoModal` component)

```javascript
const [completed, setCompleted] = useState(false);

useEffect(() => {
  const timer = setInterval(() => {
    playerRef.current?.getCurrentTime().then(data => {
      saveVideoProgress({
        videoId,
        completed,
        timeStamp: data
      });
    });
  }, 2000);

  return () => {
    clearInterval(timer);
  };
}, [videoId, completed]);
```

To seek to the previous timestamp, a callback to player `onReady` can be used. When the player becomes ready, the last updated timestamp is fetched and the video is set to that time stamp using the `seekTo` method exposed through the player ref.

(`VideoModal` component)

```javascript
const onPlayerReady = useCallback(() => {
  getVideoProgress(videoId).then(data => {
    if (data.timeStamp) {
      playerRef.current?.seekTo(data.timeStamp);
    }
  });
}, [videoId]);
```

To know when the player has finished playing a video, the `onChangeState` callback can be used. It emits a "ended" event when the video reaches the end. On receiving this event, the completed state can be set and stored subsequently in AsyncStorage.

```jsx
<YoutubeIframe
  ref={playerRef}
  play={true}
  videoId={videoId}
  height={250}
  onReady={onPlayerReady}
  onChangeState={state => {
    if (state === "ended") {
      setCompleted(true);
    }
  }}
/>
```

#### Track completion progress

This will be simple function to query completion status of each video from `AsyncStorage`.

```javascript
const getProgress = async () => {
  const total = videoSeries.length;
  let completed = 0;
  for (let i = 0; i < total; i++) {
    const videoId = videoSeries[i];
    const status = await getVideoProgress(videoId);
    if (status?.completed) {
      completed += 1;
    }
  }

  return completed / total;
};
```

This function iterates through the videoList and checks if the video was watched to completion. Using the result of this function, we can calculate progress when the modal visibility changes. (since the modal has the actual video player).

```javascript
const [progress, setProgress] = useState(0);
useEffect(() => {
  getProgress().then(p => {
    setProgress(p);
  });
}, [modalVisible]);
```

Now make a simple component to render this progress fraction as a progress bar.

```javascript
const ProgressBar = ({ progress }) => {
  const width = (progress || 0) + "%";
  return (
    <View style={{ borderWidth: 1, marginVertical: 16 }}>
      <View
        style={{
          backgroundColor: "green",
          height: 10,
          width
        }}
      />
    </View>
  );
};
```

### Final Result

<video width="134" height="295" controls="controls">
  <source src="../assets/rn_yt_3.mp4" type="video/mp4">
</video>

```javascript
import React, { useState, useEffect, useCallback, useRef } from "react";
import {
  SafeAreaView,
  Text,
  FlatList,
  Image,
  View,
  TouchableOpacity,
  Modal
} from "react-native";
import YoutubeIframe, { getYoutubeMeta } from "react-native-youtube-iframe";
import AsyncStorage from "@react-native-community/async-storage";

const videoSeries = [
  "DC471a9qrU4",
  "tVCYa_bnITg",
  "K74l26pE4YA",
  "m3OjWNFREJo"
];

const App = () => {
  const [modalVisible, showModal] = useState(false);
  const [selectedVideo, setSelectedVideo] = useState(null);
  const [progress, setProgress] = useState(0);

  const onVideoPress = useCallback(videoId => {
    showModal(true);
    setSelectedVideo(videoId);
  }, []);

  useEffect(() => {
    getProgress().then(p => {
      setProgress(p);
    });
  }, [modalVisible]);

  const closeModal = useCallback(() => showModal(false), []);

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <FlatList
        contentContainerStyle={{ margin: 16 }}
        ListHeaderComponent={
          <>
            <Text style={{ fontSize: 18, fontWeight: "bold" }}>
              100 Seconds of Code
            </Text>
            <ProgressBar progress={progress * 100} />
          </>
        }
        data={videoSeries}
        renderItem={({ item }) => (
          <VideoItem videoId={item} onPress={onVideoPress} />
        )}
        keyExtractor={item => item}
      />
      <Modal
        visible={modalVisible}
        transparent={true}
        onRequestClose={closeModal}
      >
        <VideoModal videoId={selectedVideo} onClose={closeModal} />
      </Modal>
    </SafeAreaView>
  );
};

const getProgress = async () => {
  const total = videoSeries.length;
  let completed = 0;
  for (let i = 0; i < total; i++) {
    const videoId = videoSeries[i];
    const status = await getVideoProgress(videoId);
    if (status?.completed) {
      completed += 1;
    }
  }
  return completed / total;
};

const ProgressBar = ({ progress }) => {
  const width = (progress || 0) + "%";
  return (
    <View style={{ borderWidth: 1, marginVertical: 16 }}>
      <View
        style={{
          backgroundColor: "green",
          height: 10,
          width
        }}
      />
    </View>
  );
};

const VideoItem = ({ videoId, onPress }) => {
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
        style={{ flexDirection: "row", marginVertical: 16 }}
      >
        <Image
          source={{ uri: videoMeta.thumbnail_url }}
          style={{
            width: videoMeta.thumbnail_width / 4,
            height: videoMeta.thumbnail_height / 4
          }}
        />
        <View style={{ justifyContent: "center", marginStart: 16 }}>
          <Text style={{ marginVertical: 4, fontWeight: "bold" }}>
            {videoMeta.title}
          </Text>
          <Text>{videoMeta.author_name}</Text>
        </View>
      </TouchableOpacity>
    );
  }
  return null;
};

const VideoModal = ({ videoId, onClose }) => {
  const playerRef = useRef(null);
  const [completed, setCompleted] = useState(false);

  useEffect(() => {
    const timer = setInterval(() => {
      playerRef.current?.getCurrentTime().then(data => {
        saveVideoProgress({
          videoId,
          completed,
          timeStamp: data
        });
      });
    }, 2000);

    return () => {
      clearInterval(timer);
    };
  }, [videoId, completed]);

  const onPlayerReady = useCallback(() => {
    getVideoProgress(videoId).then(data => {
      if (data.timeStamp) {
        playerRef.current?.seekTo(data.timeStamp);
      }
    });
  }, [videoId]);

  return (
    <View
      style={{
        flex: 1,
        backgroundColor: "#000000dd",
        justifyContent: "center"
      }}
    >
      <View style={{ backgroundColor: "white", padding: 16 }}>
        <Text onPress={onClose} style={{ textAlign: "right" }}>
          Close
        </Text>
        <YoutubeIframe
          ref={playerRef}
          play={true}
          videoId={videoId}
          height={250}
          onReady={onPlayerReady}
          onChangeState={state => {
            if (state === "ended") {
              setCompleted(true);
            }
          }}
        />
      </View>
    </View>
  );
};

const saveVideoProgress = ({ videoId, completed, timeStamp }) => {
  const data = {
    completed,
    timeStamp
  };

  return AsyncStorage.setItem(videoId, JSON.stringify(data));
};

const getVideoProgress = async videoId => {
  const json = await AsyncStorage.getItem(videoId);
  if (json) {
    return JSON.parse(json);
  }
  return {
    completed: false,
    timeStamp: 0
  };
};

export default App;
```
