# Sumerian Demo

## Adding a webcam to the scene

 1. Add an HTML 3D HTML entity into your scene (add a new entity, select 3D HTML)

 2. Add a video element to your HTML 3D component  
    ```<video id="videoElement" autoplay="true"></video>```

 3. Connect your webcam stream to the video element via Javascript.
    - Add a custom JS script to your HTML 3D entity
    - Edit the JS script that was created, you will need to implement the following code in the `setup()` function
        ```
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
    	}```

        If you have multiple webcams connected, you may need to pass a specific deviceId.

        ```navigator.mediaDevices.getUserMedia({video: true, deviceId: { exact: camera.deviceId }})```

        In order to find out what deviceId to use, you can use the following:
        ```navigator.mediaDevices.enumerateDevices()```

 You are ready to test your scene and see if you can view the webcam.


 ## Capturing image from the video stream (to be sent to Rekognition)


    ```// Capture an image from the video element and return it as a DataURL
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

    // Transform a dataURL (base64 encoded image) to a blob
    function imgToBlob(data) {
    	var base64Image = data.replace(/^data:image\/(png|jpeg|jpg);base64,/, '');
    	var binaryImg = atob(base64Image);
      	var length = binaryImg.length;
      	var ab = new ArrayBuffer(length);
      	var ua = new Uint8Array(ab);
      	for (var i = 0; i < length; i++) {
        	ua[i] = binaryImg.charCodeAt(i);
      	}

    	return ab;
    }```
