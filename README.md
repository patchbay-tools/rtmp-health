# rtmp-health

Health check for RTMP ingest endpoints.

A small bash wrapper around `ffprobe`: each URL is probed under a hard
timeout, and the result is printed as a line of text or a JSON object. Exit
codes follow the Nagios plugin convention, so it drops straight into Nagios,
Icinga, Zabbix external checks or a cron job.

| exit | meaning |
|------|---------|
| 0 | every stream answered with at least one audio/video stream |
| 2 | at least one stream failed or timed out |
| 3 | bad arguments or `ffprobe` missing |

## Requirements

- bash 4+
- `ffprobe` (from ffmpeg) built with RTMP support
- coreutils `timeout`

## Usage

```
rtmp-health rtmp://live.example.net/app/stream
rtmp-health -t 5 -f streams.txt
rtmp-health -j rtmp://a.example.net/live/one rtmp://b.example.net/live/two
```

Text output:

```
OK          812ms  rtmp://live.example.net/app/stream  video|h264|1280|720 audio|aac||
CRITICAL   5004ms  rtmp://backup.example.net/app/stream  timed out after 5s
CRITICAL - 1 of 2 streams down
```

JSON output (`-j`), one object per line:

```
{"url":"rtmp://live.example.net/app/stream","status":"OK","ms":812,"detail":"video|h264|1280|720 audio|aac||"}
```

A URL file takes one URL per line; blank lines and `#` comments are ignored.

## Cron

```
*/5 * * * * /usr/local/bin/rtmp-health -f /etc/rtmp-health.list >/dev/null || logger -t rtmp-health "stream down"
```
