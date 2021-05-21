Still, playing 2048? Still, playing 2048 with your hand? It is already 2021 now, use your voice to play it and feel like a boss. 😁

![](https://miro.medium.com/max/288/1*Fknp9u7t4gkCjM9bBVePPw.gif)

As the Automatic Speech Recognition (ASR) technology continuously improving, integrate a simple voice control is much easier than you thought in your App. If you don’t want to involve the AI, the basic steps for the simple voice control can be just: convert speech to text and extract the keyword from the text as a control command.

If you want to use the native library, integrate ASR in your Android App is very straightforward, you can find tutorials [here](https://developer.android.com/reference/android/speech/SpeechRecognizer) and [here](https://betterprogramming.pub/implement-continuous-speech-recognition-on-android-1dd2f4b562fd). You may notice the speech recogniser is under the android domain, but it is not included in the Android Open Source Project (AOSP) and you need the Google Mobile Service (GMS) to use this library. As the API restriction, you can also integrate the ASR with Huawei Mobile Service (HMS) [ML Kit](https://developer.huawei.com/consumer/en/doc/development/HMSCore-Guides/ml-asr-0000001050066212) in your App, you can find a nice article [here](https://medium.com/huawei-developers/the-robots-can-understand-us-now-c55a1ad073ac) to describe the integration details and the difference between these two native libraries.

In this post, I will focus on how to add the voice control feature to this popular game 2048 with these two ASR native libraries. The project I am using to demo the idea is based on this beautify compose implementation [here](https://github.com/alexjlockwood/android-2048-compose). You can find all the change in this [repo](https://github.com/pengj/android-2048-compose), feel free to play and give a comment.

## Feature Requirement

Before we dive into the implementation detail and add this feature to the existing code base, let’s sit back and think about how to add this new requirement with a design that’s easier to maintain down the road.

First, we start by understanding the needs behind this voice control feature _“user can move the tiles with their voice speech”_ and break it into more details steps:

1. Activate speech recognition when the user enables the voice control function.

2. Listen for the direction keyword from the background.

3. Once one direction keyword detected, slide numbered tiles to the direction.

4. deactivate speech recognition when the user switches off the voice function.

Once we are clear about the requirements, all the actions that can happen in the system can be gathered up. There are three main actions here: ASR, extract the direction keyword and slide tiles action. For each action, there will be a component to handle it and this will be its own single responsibility. The next step will be how to add these new components into the existing system architecture and limit the scope of the work as much as you can.

## Implementation

The direction provider will be the main interface to the voice control feature, which handles ASR and return the direction. ViewModel will get the move direction and update the game tile state. The compose game UI will be updated based on the new state. When you identify the boundary and responsibility of each component, the following architecture diagram can give a clear overview of the system you want to build.

![](https://miro.medium.com/max/576/1*b6KGE3VxuntZiXn1Kew1gw.png)

How are we going to get from that architecture design diagram to actual code? As the ASR link closely with the hardware part, after identifying the key function of ASR API, the `DirectionProvider` interface is created here to abstract the two ARS implementation and provide the core function to get the direction from other input (voice here). As both libraries need `context` to create the speech recogniser, the `destroy` function is used to release the resource when the activity `onDestory` called.

```java
interface DirectionProvider {
    fun start()
    fun stop()
    fun destroy()
}

internal class HuaweiVoiceProvider(
    private val context: Context,
    private val voiceDirectionExtractor: VoiceDirectionExtractor,
    private val onDirectionListener: (direction: Direction) -> Unit,
) : MLAsrListener, DirectionProvider

internal class VoiceProvider(
    context: Context,
    private val voiceDirectionExtractor: VoiceDirectionExtractor,
    private val onDirectionListener: (direction: Direction) -> Unit
) : RecognitionListener, DirectionProvider
```

A summary of changes when adding voice control feature is given below:

1. `GameState` class to save the current game UI state

2. Compose class `ActionSwitch` to display and control voice recognition switch

3. `DirectionProvider` for direction handling, two speech recognition class implement it, and in the future, you can also use visual service, such as the camera to detect your hand gesture

4. `VoiceDirectionExtractor` handles the keyword detection and mapping to the direction

5. `DirectionProviderFactory` to create different provider instance according to the availability without exposing the creation logic

6. Direction event call back to update the game state and handle the UI move within the `ViewModel`

## Lessons learned

1. ASR is not as good as you expected when playing the game far away from the mobile phone, using a headphone provides a better experience.

2. Higher power consumption with these ASR API as they are not intended to be used for continuous recognition, which is mentioned [here](https://developer.huawei.com/consumer/en/doc/development/HMSCore-Guides-V5/ml-asr-0000001050066212-V5#EN-US_TOPIC_0000001050710194__section699935381711) and [here](https://developer.android.com/reference/android/speech/SpeechRecognizer).

3. Start and stop the voice recogniser need to run on the main thread.

4. In the Google implementation, enable the _`EXTRA_PARTIAL_RESULTS`_ to have better speech recognition, especially when the sentence or the keyword is very short.

5. If you’re targeting SDK 23 or above, you must request run time permission [RECORD_AUDIO](https://developer.android.com/reference/android/Manifest.permission#RECORD_AUDIO) inside your App because it may pose a risk to the user’s privacy.

6. Better to set the `FLAG_KEEP_SCREEN_ON` when enable the voice control to prevent the screen from going to sleep.

## What can be improved

As this is just a quick Proof of Concept (POC) to demo how easy to add the voice control feature in a simple App, there are lots of places that can be improved for the code and design:

1. Handle the life cycle of voice recogniser within the view model.

2. Move the keyword string from hardcode text to string resource.

3. Handle language detection or based on the system setting.

4. Only request permission when the user wants to use the voice control.

5. Voice feature can be a feature module and installed on demand.

With the voice control enabled, this also provides an opportunity to have more way to play the 2048 game. For example, you can say more than one direction within a sentence, the game can decide to apply one or all within one round then the score can be doubled when the tile combined. Feel free to fork the [repo](https://github.com/pengj/android-2048-compose) and explore new ideas.
