# Software-Design-Architecture
High Level Design

![image](https://user-images.githubusercontent.com/115500959/195161832-b282ae91-c571-499f-92fe-64e83215598d.png)

### Video Live Streaming Architechture And Details explanation
<br>
We just published a YouTube video that explains how live streaming (YouTube live, TikTok live, Twitch) works.<br>
 
You can find the video link at the end of the post.<br>
 
If you prefer text, you can keep reading:<br>
 
Live streaming is challenging because the video content is sent over the internet in near real-time. Video processing is compute-intensive. 
Sending a large volume of video content over the internet takes time. These factors make live streaming challenging.<br>

The diagram below explains what happens behind the scenes to make this possible.<br>
 
Step 1: The streamer starts their stream. The source could be any video and audio source wired up to an encoder <br>
 
Step 2: To provide the best upload condition for the streamer, most live streaming platforms provide point-of-presence servers worldwide. 
The streamer connects to a point-of-presence server closest to them.<br>
 
Step 3: The incoming video stream is transcoded to different resolutions, and divided into smaller video segments a few seconds in length.<br>
 
Step 4: The video segments are packaged into different live streaming formats that video players can understand. 
The most common live-streaming format is HLS, or HTTP Live Streaming.<br>
 
Step 5: The resulting HLS manifest and video chunks from the packaging step are cached by the CDN. <br>
 
Step 6: Finally, the video starts to arrive at the viewer’s video player.<br>
 
Step 7-8: To support replay, videos can be optionally stored in storage such as Amazon S3.<br>
