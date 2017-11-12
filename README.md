# m3u8-parser
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.lindstrom/m3u8-parser/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.lindstrom/m3u8-parser)

A simple HLS playlist parser for Java.

The goal of this project was to implement parsers and a consistent Java object model
according to [RFC 8216 HTTP Live Streaming](https://tools.ietf.org/html/rfc8216).

This parser is very similar to iHeartRadio's [open-m3u8](https://github.com/iheartradio/open-m3u8). The main differences are:
 * m3u8-parser does not try to validate playlists. You are responsible for creating valid playlists.
 * m3u8-parser uses java.util.Optional instead of `null`.
 * m3u8-parser uses [Immutables](https://immutables.github.io/) to generate all builders.
 * The parser objects are thread safe & re-usable and could used as a singleton (like Jackson's ObjectMapper).
 * m3u8-parser requires Java 8 or later.

## Artifacts
Maven:
```xml
<dependency>
    <groupId>io.lindstrom</groupId>
    <artifactId>m3u8-parser</artifactId>
    <version>0.3</version>
</dependency>
```

Gradle:
```
compile 'io.lindstrom:m3u8-parser:0.3'
```

## Usage

### Create master playlist
```java
MasterPlaylist playlist = MasterPlaylist.builder()
                .version(4)
                .independentSegments(true)
                .addAlternativeRenditions(AlternativeRendition.builder()
                        .type(MediaType.AUDIO)
                        .name("Default audio")
                        .groupId("AUDIO")
                        .build())
                .addVariants(
                        Variant.builder()
                                .addCodecs("avc1.4d401f", "mp4a.40.2")
                                .bandwidth(900000)
                                .uri("v0.m3u8")
                                .build(),
                        Variant.builder()
                                .addCodecs("avc1.4d401f", "mp4a.40.2")
                                .bandwidth(900000)
                                .uri("v1.m3u8")
                                .resolution(1280, 720)
                                .build())
                .build();

MasterPlaylistParser parser = new MasterPlaylistParser();
System.out.println(parser.writePlaylistAsString(playlist));
```

This code should produce the following playlist:
```
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-INDEPENDENT-SEGMENTS
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="AUDIO",NAME="Default audio"
#EXT-X-STREAM-INF:BANDWIDTH=900000,CODECS="avc1.4d401f,mp4a.40.2"
v0.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=900000,CODECS="avc1.4d401f,mp4a.40.2",RESOLUTION=1280x720
v1.m3u8
```

### Parse master playlist
```java
MasterPlaylistParser parser = new MasterPlaylistParser();

// Parse playlist
MasterPlaylist playlist = parser.readPlaylist(Paths.get("path/to/master.m3u8"));

// Update playlist version
MasterPlaylist updated = MasterPlaylist.builder()
                                        .from(playlist)
                                        .version(2)
                                        .build();

// Write playlist to standard out
System.out.println(parser.writePlaylistAsString(updated));
```

### Parse media playlist
```java
MediaPlaylistParser parser = new MediaPlaylistParser();

// Parse playlist
MediaPlaylist playlist = parser.readPlaylist(Paths.get("path/to/media-playlist.m3u8"));

// Update playlist version
MediaPlaylist updated = MediaPlaylist.builder()
                                     .from(playlist)
                                     .version(2)
                                     .build();

// Write playlist to standard out
System.out.println(parser.writePlaylistAsString(updated));
```

## Supported tags
The following tags should be fully supported:
```
EXTM3U
EXT-X-VERSION
EXTINF
EXT-X-BYTERANGE
EXT-X-DISCONTINUITY
EXT-X-KEY
EXT-X-MAP
EXT-X-PROGRAM-DATE-TIME
EXT-X-TARGETDURATION
EXT-X-MEDIA-SEQUENCE
EXT-X-ENDLIST
EXT-X-PLAYLIST-TYPE
EXT-X-I-FRAMES-ONLY
EXT-X-MEDIA
EXT-X-STREAM-INF
EXT-X-I-FRAME-STREAM-INF
EXT-X-INDEPENDENT-SEGMENTS
```

The following tags are currently not implemented:
```
EXT-X-SESSION-DATA
EXT-X-SESSION-KEY
EXT-X-DISCONTINUITY-SEQUENCE
EXT-X-DATERANGE
EXT-X-START
```

