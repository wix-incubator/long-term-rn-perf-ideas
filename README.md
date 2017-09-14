# Discussion

> Long term React Native performance improvement ideas and future architecture research

The purpose of this repo is to discuss some ideas we've been thinking about regarding some long *long* term potential improvements to React Native. Nothing is solid here, just interesting directions worth exploring. Some of them are completely wild and may turn out to be total nonsense.

## Prioritization in the bridge

The JavaScript thread and React Native bridge are both bottlenecks in a busy React Native app. Events originating from native that the JavaScript thread must handle are queued up on the bridge and aren't handled until the queue clears up.

Some events that are sent on the bridge are more important than others. For example, when scrolling down a scroll view, new rows must be rendered and the native scroll event is what triggeres these renders. Another example is touch events that also originate from native and must wait in line to be handled by JavaScript.

Other events have lower priority. For example, an HTTP fetch response is queued on the bridge to be handled by JavaScript after the server responds. The response already takes a long time to arrive due to network latency, so waiting a few more miliseconds on the bridge queue doesn't make a big difference.

Bridge messages are currently handled in the order they arrive. Adding priorities to the bridge queue can go a long way in handling important events faster. Canceling bridge messages is also an important feature. Consider the scroll view example, where an upcoming row is waiting in line to be rendered, but the user ultimately decides to scroll in the other direction and the upcoming row is no longer needed.

This mechanism can complement nicely the render prioritization introduced in react fiber.

## Multiple JSCore instances to take advantage of multiple cores

## Parallel threads in a single JSCore instance

## Doing JIT in build time
