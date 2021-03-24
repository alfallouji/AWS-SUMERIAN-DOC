<video id="videoElement" autoplay="true"></video>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/posenet"></script>

<script>

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

  const net = await posenet.load();
  var image = captureImage(video);
  const pose = await net.estimateSinglePose(image, {
    flipHorizontal: false
  });

</script>
