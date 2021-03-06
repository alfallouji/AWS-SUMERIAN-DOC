# Sumerian Demo

## Adding a webcam to the scene

 1. Add an HTML 3D HTML entity into your scene (add a new entity, select 3D HTML)

 2. Add a video element to your HTML 3D component  
    ```<video id="videoElement" autoplay="true"></video>```

 3. Connect your webcam stream to the video element via Javascript.
    - Add a custom JS script to your HTML 3D entity
    - Edit the JS script that was created, you will need to implement the following code in the `setup()` function
```javascript
var video = document.querySelector("#video");
if (navigator.mediaDevices.getUserMedia) {       
     navigator.mediaDevices.getUserMedia({video: true})
     .then(function(stream) {
          console.log('Attaching video stream to the video element');
          video.srcObject = stream;
     })
     .catch(function(error) {
          console.log("Couldn't attach the video stream. Caught following error", error);
     });
}
```

        If you have multiple webcams connected, you may need to pass a specific deviceId.

```javascript
     navigator.mediaDevices.getUserMedia({video: true, deviceId: { exact: camera.deviceId }})
```

        In order to find out what deviceId to use, you can use the following:
```javascript
     navigator.mediaDevices.enumerateDevices()
```

 You are ready to test your scene and see if you can view the webcam.


 ## Capturing image from the video stream (to be sent to Rekognition)

 1. The following function will allow you to capture an image from the video feed:

```javascript
    /**
     * Capture an image from the video element and return it as a DataURL
     * 
     * @param video HTMLElement Video HTML Element
     * @return string base64 encoded string of the captured image
     */
    function captureImage(video) {
    	var scale = 1;
     var canvas = document.createElement("canvas");
    	canvas.width = video.videoWidth * scale;
    	canvas.height = video.videoHeight * scale;
    	canvas.getContext('2d').drawImage(video, 0, 0, canvas.width, canvas.height);
    	document.getElementById('div-image').innerHTML = '';
    	document.getElementById('div-image').appendChild(canvas);

    	return canvas.toDataURL('image/jpeg');
    }
```


## Using Amazon Rekognition to search for faces
 1. To configure the Sumerian Scene to access AWS, you can follow this tutorial:
 https://docs.aws.amazon.com/sumerian/latest/userguide/scene-aws.html

 2. Amazon Rekognition allows you to search for a face within a collection.  
 https://docs.aws.amazon.com/rekognition/latest/dg/API_SearchFacesByImage.html
 
 The SearchFacesByImage can receive a blob or an arraybuffer as an input for the image (it can also receive an object stored in a S3 bucket):
 
```javascript
    /**
     * Transform a dataURI (base64 encoded image) to a blob
     * 
     * @param string data Base64 encoded string of an image
     * @return ArrayBuffer ArrayBuffer containing the image
     */
    function dataURItoBlob(data) {
    	var base64Image = data.replace(/^data:image\/(png|jpeg|jpg);base64,/, '');
    	var binaryImg = atob(base64Image);
      	var length = binaryImg.length;
      	var ab = new ArrayBuffer(length);
      	var ua = new Uint8Array(ab);
      	for (var i = 0; i < length; i++) {
        	ua[i] = binaryImg.charCodeAt(i);
      	}

    	return ab;
    }
```

Here is a snippet of code on how to invoke Rekognition and do a searchByFace.

```javascript
var collectionId = "my-aws-rekognition-collection-id";
var imageData = captureImage(video);
var imageBlob = dataURItoBlob(imageData);

// Send it to Rekognition now
var params = {
    CollectionId: collectionId, 
    FaceMatchThreshold: 90, 
    Image: {
        Bytes: imageBlob
    }, 
    MaxFaces: 5
};

var rekognition = new AWS.Rekognition();

var response = rekognition.searchFacesByImage(params, function(err, data) {
    if (err) {
        console.log(err); // an error occurred
        ctx.worldData.detectionInProgress = false;	
        ctx.transitions.failure();
        
    }
    // successful response
    else {
        console.log(data);       	
        if (data.FaceMatches.length == 0) {
            // We didnt find anyone matching in Rekogniton 
            ctx.worldData.detectionInProgress = false;		
            console.log("Couldnt find any face for this person in Rekognition");
            ctx.transitions.failure();
        } else {
            // We found someone
            var externalImageId = data.FaceMatches[0].Face.ExternalImageId.toLowerCase();
            
            console.log(externalImageId);
            ctx.worldData.externalImageId = externalImageId;
            ctx.transitions.success();
        }
    }
});```
