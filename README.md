# Discussion

> Long term React Native performance improvement ideas and future architecture research

The purpose of this repo is to discuss some ideas we've been thinking about regarding some long *long* term potential improvements to React Native. Nothing is solid here, just interesting directions worth exploring. Some of them are completely wild and may turn out to be total nonsense.

## Prioritization on the bridge

The JavaScript thread and React Native bridge are both bottlenecks in a busy React Native app. Events originating from native that the JavaScript thread must handle are queued up on the bridge and aren't handled until the queue clears up.

Some events that are sent on the bridge are more important than others. For example, when scrolling down a scroll view, new rows must be rendered and the native scroll event is what triggeres these renders. Another example is touch events that also originate from native and must wait in line to be handled by JavaScript.

Other events have lower priority. For example, an HTTP fetch response is queued on the bridge to be handled by JavaScript after the server responds. The response already takes a long time to arrive due to network latency, so waiting a few more miliseconds on the bridge queue doesn't make a big difference.

Bridge messages are currently handled in the order they arrive. Adding priorities to the bridge queue can go a long way in handling important events faster. Canceling bridge messages is also an important feature. Consider the scroll view example, where an upcoming row is waiting in line to be rendered, but the user ultimately decides to scroll in the other direction and the upcoming row is no longer needed.

This mechanism can complement nicely the render prioritization introduced in react fiber.

## Multiple JSCore instances to take advantage of multiple cores

The fact that JavaScript is single threaded often causes congestion in React Native apps. Pure native apps can spawn up as many background threads as needed and take advantage of the multiple CPU cores available in mobile devices today. Devices with 8 cores are not uncommon and the inability to make use of these extra cores is a shame.

In complex apps, there are often different sections of the app that are fairly separate and maintain their own state and views. We can view these sections as multiple "micro-services" in the frontend. Architecting large apps this way can add the equivalent flexibility of locating mutiple micro-services on multiple machines.

In our case, multiple machines can be equivalent to mutliple instances of JSCore. Each of these VMs will naturally have its own separate JavaScript heap and separate execution thread that can run on its own core.

Communication between JSCore instances can be done using a dedicated native module that implements a serializable messaging interface or event bus. React components can also be shared between instances by wrapping them in native React root views.

The main downside of this approach is the memory overhead. An empty JSCore instance should cost about 5 MB of RAM. If we're planning to parse our entire JavaScript bundle on every instance, with the hundreds of dependencies React Native has, the number can grow quite a bit. This might not be that bad if we're only going to spin up 2-3 instances. We don't have that many cores anyways.

## Parallel threads in a single JSCore instance

Traditionally, JavaScript has a single thread execution environment. Constructs such as web workers are entities with separate memory spaces which make sharing data difficult and slow through serialization.

Lately, we've been hearing about various models for concurrency in JavaScript under the same memory space. The WebKit blog published a rather long [post](https://webkit.org/blog/7846/concurrent-javascript-it-can-work/) on this topic.

If these techniques can indeed become operable, 

## Doing JIT in build time

## Asynchronous native 
