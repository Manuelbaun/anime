
# Why
I have added some changes to the library to support, what I needed for a uni project. It mainly concerned the timeline of animejs. It worked fine for me, but might cause some problem for others.

I have modified the code to enable add animation dynamically, while running or in pause mode, so that the library would not reset the timeline. Also you can continue the animation after new animation where added.


### How to use
#### Setup timeline
```js
const tl = anime.timeline({
    easing: "easeInOutQuad",
    duration: 200,          // specify you default duration
    autoplay: false,        // set autoplay off
    shouldReset: false,     // when new animation are added, the timeline wont be resetted
    useDeltaTime: true,     // activates a different progress calculation of the timeline
});
```

#### Use in code

Add some animation and await the continue promise. Then add more and it will and await again. It will continue, where it last finished.
```js
tl.add({
    targets: 'rect',
    duration: 200,
    fill: 'red',
}).add({
    targets: 'group',
    duration: 800,
    translateY: 600,
});

// pause after 500ms
setTimeout(()=>{
    tl.pause()
    // continue after 500 ms
    setTimeout(() => tl.continue(), 500)
}, 500);


// this will run the animation of 1000 ms
// note, if the timeline is paused, this promise wont be fulfilled and will wait until it is finished
tl.play(); 
await tl.finished


tl.add({
    targets: 'rect',
    fill: 'blue',
    duration: 200,
}).add({
    targets: 'group',
    duration: 500,
    opacity: 0,
    translateX: 200,
});

// continues from where it last stopped and not from the start
await tl.continue(); 
```




---
### Changes
This section shows the changes in the code

##### 1. shouldReset: No Reset, when animation are added
If  `shouldReset` is `false`, when new animation are added, the timeline wont be set to the beginning.
This enables dynamically add new animation, without resetting the timeline.


```js
1375 if (params.shouldReset == undefined || params.shouldReset == true) {
1376    tl.seek(0);
1377    tl.reset();
1378 }
```


##### 2. useDeltaTime: Progress Calculation
The progress calculation behaved wired, when playing with the animation speed, while the animation was running. 
When the param `useDeltaTime` is set to `true`, the progress gets calculated, what I think made most sense to me, for my use case. If it is not set, it defaults to the original calculations



```js
1150 if (instance.useDeltaTime) {
1151    // fixed calculation 1000 ms / 60 fps = 16.6
1152    const deltaTime = 1000 / 60;
1153    const progress = instance.currentTime + deltaTime * anime.speed;
1154    setInstanceProgress(progress);
1155 } else {
1156    // old calculation
1157    const progress = (now + (lastTime - startTime)) * anime.speed;
1158    setInstanceProgress(progress);
1159 }

```


##### 3. continue method
The problem with the `play` method was, once the animation was finished and new animation were added, the `play`-method would reset the whole timeline. The continue method is almost the same as the `play` method, just removed the `resetTime()`-function call and the resetting of the individual animations.


```js
1201 instance.continue = function(skip = true) {
1202
1203    if (skip) {
1204        instance.seek(instance.duration)
1205    }
1206
1207
1208    if (!instance.paused) {
1209        return;
1210    }
1211    
1212    if (instance.completed) {
1213        var direction = instance.direction;
1214        instance.passThrough = false;
1215        instance.paused = true;
1216        instance.began = false;
1217        instance.loopBegan = false;
1218        instance.changeBegan = false;
1219        instance.completed = false;
1220        instance.changeCompleted = false;
1221        instance.reversePlayback = false;
1222        instance.reversed = direction === "reverse";
1223        instance.remaining = instance.loop;
1224        children = instance.children;
1225        childrenLength = children.length;
1226        const aniProgress = instance.reversed ? instance.duration : 0;
1227        setAnimationsProgress(aniProgress);
1228    }
1229    instance.paused = false;
1230    activeInstances.push(instance);
1231    engine();
1232    return instance.finished;
1233 };
```