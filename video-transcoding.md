Below is a **practical, production-style guide** to using **FFmpeg for video transcoding from Go**, including simple usage, progress tracking, and best practices.

---

## 1️⃣ Prerequisites

### Install FFmpeg

Make sure FFmpeg is available on your system:

```bash
ffmpeg -version
```

If not installed:

* **macOS**: `brew install ffmpeg`
* **Ubuntu**: `sudo apt install ffmpeg`
* **Windows**: Download from ffmpeg.org and add to PATH

---

## 2️⃣ Basic Transcoding in Go (Using `os/exec`)

### Example: Convert MP4 → H.264 + AAC

```go
package main

import (
	"log"
	"os/exec"
)

func main() {
	cmd := exec.Command(
		"ffmpeg",
		"-i", "input.mov",
		"-c:v", "libx264",
		"-preset", "fast",
		"-crf", "23",
		"-c:a", "aac",
		"-b:a", "128k",
		"output.mp4",
	)

	cmd.Stdout = nil
	cmd.Stderr = nil

	err := cmd.Run()
	if err != nil {
		log.Fatalf("transcoding failed: %v", err)
	}
}
```

### Key Options

| Option         | Meaning                                |
| -------------- | -------------------------------------- |
| `-c:v libx264` | H.264 encoder                          |
| `-crf 23`      | Quality (lower = better, 18–28 common) |
| `-preset fast` | Speed vs compression                   |
| `-c:a aac`     | Audio codec                            |

---

## 3️⃣ Capture FFmpeg Output & Errors

FFmpeg writes progress to **stderr**, not stdout.

```go
cmd := exec.Command("ffmpeg", "-i", "input.mp4", "output.mp4")

stderr, _ := cmd.StderrPipe()
cmd.Start()

scanner := bufio.NewScanner(stderr)
for scanner.Scan() {
	log.Println(scanner.Text())
}

cmd.Wait()
```

---

## 4️⃣ Transcoding with Progress Tracking (%)

### Step 1: Get Video Duration

```go
func getDuration(file string) float64 {
	cmd := exec.Command(
		"ffprobe",
		"-v", "error",
		"-show_entries", "format=duration",
		"-of", "default=noprint_wrappers=1:nokey=1",
		file,
	)

	out, _ := cmd.Output()
	duration, _ := strconv.ParseFloat(strings.TrimSpace(string(out)), 64)
	return duration
}
```

---

### Step 2: Parse Progress

```go
func transcodeWithProgress(input, output string) {
	duration := getDuration(input)

	cmd := exec.Command(
		"ffmpeg",
		"-i", input,
		"-c:v", "libx264",
		output,
	)

	stderr, _ := cmd.StderrPipe()
	cmd.Start()

	scanner := bufio.NewScanner(stderr)
	for scanner.Scan() {
		line := scanner.Text()

		if strings.Contains(line, "time=") {
			t := parseTime(line)
			percent := (t / duration) * 100
			log.Printf("Progress: %.2f%%\n", percent)
		}
	}

	cmd.Wait()
}
```

---

### Time Parsing Helper

```go
func parseTime(line string) float64 {
	re := regexp.MustCompile(`time=(\d+):(\d+):(\d+\.\d+)`)
	m := re.FindStringSubmatch(line)
	if len(m) != 4 {
		return 0
	}

	h, _ := strconv.ParseFloat(m[1], 64)
	mm, _ := strconv.ParseFloat(m[2], 64)
	s, _ := strconv.ParseFloat(m[3], 64)

	return h*3600 + mm*60 + s
}
```

---

## 5️⃣ Common Transcoding Profiles

### 🎥 1080p MP4 (Web)

```bash
ffmpeg -i input.mp4 -vf scale=1920:1080 \
-c:v libx264 -preset slow -crf 22 \
-c:a aac -b:a 128k output.mp4
```

---

### 📱 Mobile-Friendly (Fast Encode)

```bash
ffmpeg -i input.mp4 -c:v libx264 -preset veryfast -crf 28 output.mp4
```

---

### 🔹 HLS Streaming

```bash
ffmpeg -i input.mp4 \
-c:v libx264 -c:a aac \
-hls_time 6 -hls_playlist_type vod \
index.m3u8
```

---

## 6️⃣ Running FFmpeg Safely in Production

### ✔ Use Context for Timeouts

```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Minute)
defer cancel()

cmd := exec.CommandContext(ctx, "ffmpeg", "-i", "in.mp4", "out.mp4")
cmd.Run()
```

---

### ✔ Limit CPU Usage

```bash
-threads 2
```

---

### ✔ Validate Input

```go
if _, err := os.Stat(input); err != nil {
	log.Fatal("input file missing")
}
```

---

## 7️⃣ When to Use a Go FFmpeg Wrapper

If you want **less shell parsing**, consider:

* `github.com/u2takey/ffmpeg-go`
* `github.com/xfrr/goffmpeg`

Example:

```go
ffmpeg_go.
	Input("input.mp4").
	Output("output.mp4", ffmpeg_go.KwArgs{"c:v": "libx264"}).
	Run()
```

---

## 8️⃣ Architecture Tip (Based on Your Interests)

Given your recent focus on **distributed systems and Go**, a scalable setup looks like:

```
API → Job Queue → Transcoding Workers (FFmpeg) → Object Storage
```

Each worker runs isolated FFmpeg jobs with CPU limits.

---

If you want, I can:

* Build a **full transcoding microservice in Go**
* Add **HLS + adaptive bitrate**
* Show **Docker + Kubernetes FFmpeg workers**
* Optimize for **speed vs quality tradeoffs**

Just tell me what direction you want to go 🚀
