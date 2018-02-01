Media Optimization
==================

Optimize Video with ffmpeg
--------------------------
Calculate the bitrate you need by dividing 1 GB by the video length in seconds. So, for a video of length 16:40 (1000 seconds), use a bitrate of 1000000 bytes/sec:

.. code-block:: sh

	ffmpeg -i input.mp4 -b 1000000 output.mp4

Vary the CRF between around 18 and 24 — the lower, the higher the bitrate.

.. code-block:: sh

	ffmpeg -i input.mp4 -vcodec libx264 -crf 20 output.mp4 

Scale the video, keep aspect ratio so that height is adjusted to fit: ``-vf scale=1280:-2,setsar=1:1``

Audio options: ``-ac 1 -b:a 64k``
