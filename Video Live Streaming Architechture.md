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


# HTTP Live Streaming (HLS) in Real-Time Scenarios

HTTP Live Streaming (HLS) is a protocol used for delivering live and on-demand media content over the internet. It segments the media into short HTTP-based file segments, facilitating adaptive bitrate streaming and ensuring robust performance across varying network conditions.

## Overview of HLS Workflow:

1. **Encoding and Segmenting**:
   - **Source Encoding**: Live video or pre-recorded content is encoded into multiple quality levels (bitrates/resolutions) using codecs like H.264 or H.265.
   - **Segmentation**: Encoded content is segmented into short-duration files (2-10 seconds) in formats like MPEG-2 TS or fragmented MP4.

2. **Manifest File (Playlist)**:
   - **Master Playlist**: Contains references to different bitrate variants (streams) and their respective playlist URLs.
   - **Variant Playlists**: Each variant playlist lists URLs for individual video and audio segments corresponding to specific bitrates/resolutions.

3. **Client Request and Playback**:
   - **Initialization**: Clients (web browsers, mobile apps) request the master playlist from the server via HTTP.
   - **Playlist Selection**: Based on network conditions, clients select an appropriate variant playlist from the master playlist.
   - **Segment Fetching**: Clients fetch video and audio segments sequentially from URLs provided in the selected variant playlist.
   - **Adaptive Bitrate Streaming**: Clients switch between bitrate variants dynamically to ensure smooth playback without interruptions.

4. **Live Updates**:
   - **Segment Updates**: For live streams, new segments are continuously added to the playlist as the event progresses.
   - **Playlist Updates**: Master playlist is updated periodically to reflect the availability of new segments, facilitating seamless transition for clients.

5. **Buffering and Latency**:
   - **Buffering**: Clients buffer a few segments ahead to handle network fluctuations and minimize playback interruptions.
   - **Latency**: HLS introduces some delay (seconds) due to segment buffering and client-side processing; efforts like low-latency HLS aim to reduce this for real-time applications.

## Advantages of HLS:

- **Adaptive Streaming**: Smooth playback across varying network conditions with adaptive bitrate streaming.
- **Compatibility**: Widely supported on platforms (iOS, Android, web) due to standard HTTP protocols.
- **Scalability**: Easily scalable using Content Delivery Networks (CDNs) for global distribution.
- **DVR Functionality**: Supports DVR-like functionalities (pause, rewind) for live streams by buffering segments.

## Real-Time Scenarios:

- **Live Sports Events**: Broadcasting live sports events with adaptive streaming for diverse viewer devices.
- **News Broadcasting**: Continuous live news coverage with seamless quality adjustments.
- **Online Gaming**: Streaming live gaming sessions with low latency and adaptive bitrate for optimal viewing.

HTTP Live Streaming (HLS) remains a preferred choice for delivering live and on-demand media content due to its reliability, scalability, and adaptive streaming capabilities.

