
# Group Video Conversation


 _**Applies to:** Skype for Business 2015_

## Starting a group video conversation

1. As a first step we need to create a conversation

    ```js
        var conversation = application.conversationsManager.createConversation();
    ```

1. Now we are adding a listener to display our own video when the camera is turned on.
When the listener is triggered, we can assign a DOM node as the container for the video.

    ```js
        conversation.selfParticipant.video.state.when('Connected', function () {
            // video is availabe ... lets assign a container.
            conversation.selfParticipant.video.channels(0).stream.source.sink.format('Stretch'); // formats include: 'Stretch', 'Fit' and 'Crop'
            conversation.selfParticipant.video.channels(0).stream.source.sink.container(/* pass in a DOM node such as a DIV */);
            // the video will be rendered automatically
        });
    ```
1. To render the video of the other participants in the conversation we add another listener that notifies us when
another participant joins the conversation.

    ```js
        conversation.participants.added(function (person) {
            // person.displayName() has joined the conversation

            // lets add another listener for the joined person that notifies us when they add video
            person.video.state.when('Connected', function () {
                // video is availabe ... lets assign a container.
                person.video.channels(0).stream.source.sink.format('Stretch'); // formats include: 'Stretch', 'Fit' and 'Crop'
                person.video.channels(0).stream.source.sink.container(/* pass in a DOM node such as a DIV */);

                // now we can render it
                person.video.channels(0).isStarted(true);

                // NOTE: .isStarted() only needs to be called for remote participants in group conversations
                // it dictates wether or not the participant's video should be rendered
            });
        });
    ```

1. Last but not least we can add participants to the conversation and start the video conversation.

    ```js
        conversation.participants.add('<participant1.id>');
        conversation.participants.add('<participant2.id>');

        // starting the video conference
        conversation.videoService.start().then(null, function (error) {
            // handle error
        });
    ```

## Ending the conversation

 ```js
    conversation.leave().then(function () {
        // conversation ended
    }, function (error) {
        // handle error
    }).then(function () {
        // clean up operations
    });
```

## Starting a group video conversation

The application object exposes a conversationsManager object which we can use to create new group conversation by calling createConvresation().  After creation of the conversation object, it is helpful to setup a few event listeners for when we are connected to video, added participants, when participants are connected to video, and when we disconnect from the conversation.

When either the selfParticipant or other persons are conencted to video we also need to setup the DOM element where the video should be displayed.  This configuration can be handled by getting access to the sink object by walking from the person object to the video modality to the channels collection choosing the first, or channels(0), which gives us access to the stream object which has a source object which finally points us to the sink object.  The sink object has a format property that can accept video formatting options such as Stretch, Fit, and Crop.  The sink object also exposes a container property where we can provide a DOM element where the video will be inserted.

We can add participants to the conversation by calling add(...) providing a SIP URI on the participants collection of the conversation object.  We can use the videoService on the conversation object and call start() to initate the call.

After the conversation and video modality are established we can begin communicating with the remote parties.  When finished click the end button to terminate the conversation.


### Start a group video conversation

1. Start group video conversation, and track participant events 

  ```js
    var conversationsManager = application.conversationsManager;
    var id = content.querySelector('.id').value;
    var id2 = content.querySelector('.id2').value;
    conversation = conversationsManager.createConversation();

    function setupContainer(person, size, videoDiv) {
        person.video.channels(0).stream.source.sink.format('Stretch');
        person.video.channels(0).stream.source.sink.container(videoDiv);
    }

    listeners.push(conversation.selfParticipant.video.state.when('Connected', function () {
        // set up local video container
        setupContainer(conversation.selfParticipant, 'large', content.querySelector('.selfVideoContainer'));

        listeners.push(conversation.participants.added(function (person) {
            // person.displayName() has joined the conversation
            listeners.push(person.video.state.when('Connected', function () {
                // set up remote video container
                if (Object.keys(videoMap).length === 1) {
                    videoMap[person.displayName()] = 2;
                    setupContainer(person, 'large', content.querySelector('.remoteVideoContainer2'));
                }
                else {
                    videoMap[person.displayName()] = 1;
                    setupContainer(person, 'large', content.querySelector('.remoteVideoContainer1'));
                }
                person.video.channels(0).isStarted(true);
            }));
        }));
    }));

    listeners.push(conversation.state.changed(function (newValue, reason, oldValue) {
        if (newValue === 'Disconnected' && (oldValue === 'Connected' || oldValue === 'Connecting')) {
            // Conversation ended
        }
    }));

    conversation.participants.add(id);
    conversation.participants.add(id2);
    conversation.videoService.start().then(null, function (error) {
        // handle error
    });
  ```

2. **Advanced**: Track remote participant video state

    ```js
    listeners.push(person.video.channels(0).isVideoOn.when(true, function () {
        // person.displayName() started streaming their video
    }));
    listeners.push(person.video.channels(0).isVideoOn.when(false, function () {
       // person.displayName() stopped streaming their video
    }));
  ```

3. End the conversation

  ```js
    conversation.leave().then(function () {
        // conversation ended
    }, function (error) {
        // handle error
    }).then(function () {
        // clean up operations
    });
  ```
