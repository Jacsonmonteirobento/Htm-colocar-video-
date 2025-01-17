<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style type="text/css">

        body {
            margin: 0;
        }

        video#video {
            display: block;
        }

        div#time, div#frames, div#binary {
            font-family: "Arial", sans-serif;
            font-size: 60px;
            font-weight: bold;
            float: left;
            margin-left: 40px;
        }

        div#binary {
            font-family: "Courier New", sans-serif;
            font-weight: bold;
        }

        #btn {
            float: left;
            font-size: 20px;
        }

    </style>
</head>
<body>
<video id="video" src="http://s3-eu-west-1.amazonaws.com/singuerinc--cdn/blog-files/day-022/test_film_counter.mp4" width="1282" height="720" autoplay></video>
<button id="btn">Play/Pause</button>
<div id="time"></div>
<div id="frames"></div>
<div id="binary"></div>

<script type="text/javascript" src="stats.min.js"></script>
<script type="text/javascript">
    var stats = new Stats();
    stats.setMode(0); // 0: fps, 1: ms

    // Align top-left
    stats.domElement.style.position = 'absolute';
    stats.domElement.style.right = '0px';
    stats.domElement.style.top = '0px';

    document.body.appendChild(stats.domElement);


    (function () {
        var lastTime = 0;
        var vendors = ['ms', 'moz', 'webkit', 'o'];
        for (var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {
            window.requestAnimationFrame = window[vendors[x] + 'RequestAnimationFrame'];
            window.cancelRequestAnimationFrame = window[vendors[x] +
                    'CancelRequestAnimationFrame'];
        }
        if (!window.requestAnimationFrame)
            window.requestAnimationFrame = function (callback) {
                var currTime = new Date().getTime();
                var timeToCall = Math.max(0, 16 - (currTime - lastTime));
                var id = window.setTimeout(function () {
                            callback(currTime + timeToCall);
                        },
                        timeToCall);
                lastTime = currTime + timeToCall;
                return id;
            };

        if (!window.cancelAnimationFrame)
            window.cancelAnimationFrame = function (id) {
                clearTimeout(id);
            };
    }());

    var btn = document.getElementById('btn'),
            video = document.getElementById('video'),
            canvas = document.createElement('canvas'),
            time = document.getElementById('time'),
            frames = document.getElementById('frames'),
            binary = document.getElementById('binary'),
            ctx = canvas.getContext('2d'),
            i, lastTime,
            paused = true,
            videoWidth = 1282,
            videoHeight = 720,
            videoFrameRate = 25,
            numPixToRead = 12;

    canvas.width = 1;
    canvas.height = 720;

    var draw = function () {

        stats.begin();

        // draw the top-right 1px x 48px of the video in a tmp canvas
        ctx.drawImage(video, videoWidth - 1, 0, 1, numPixToRead * 4, 0, 0, 1, numPixToRead * 4);

        // get pixels data
        var pixData = ctx.getImageData(0, 0, 1, numPixToRead).data;

        var binStr = '';
        for (var i = 0, n = pixData.length; i < n; i += 4) {
            //the pixel data has 4 values by pixel read (r,g,b,a)

            // after the video compression the pixels
            // aren't black (0) or white (255) so we need to
            // separate them by luminosity and convert in 0s and 1s
            var binNum = pixData[i] > 127 ? 1 : 0;

            // store binary num in reverse order
            binStr = binNum + '' + binStr;
        }

        // the FRAME number
        // convert binary string into decimal
        var frame = parseInt(binStr, 2);

        var t = Math.floor(frame / videoFrameRate);
        var secs = Math.floor(t / 60) + ':' + (t % 60);

        binary.innerText = binStr;
        frames.innerText = frame.toString();
        time.innerText = secs;

        stats.end();
    };

    var animate = function () {
        draw();
        requestAnimationFrame(animate);
    };

    video.addEventListener('loadeddata', function () {
        paused = false;
        animate();
    }, false);

    btn.addEventListener('click', function (e) {
        if (paused) {
            paused = false;
            video.play();
        }
        else {
            paused = true;
            video.pause();
        }
    });

</script>
</body>
</html>
