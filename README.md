# thumbnail-video
Thumbnail and preview video
package com.example.thumbnail_video;

import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.FFmpegFrameRecorder;
import org.bytedeco.javacv.Frame;
import org.bytedeco.javacv.Java2DFrameConverter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.nio.file.DirectoryStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.UUID;

@SpringBootApplication
public class ThumbnailVideoApplication {
	
	public static void main(String[] args) {
		DirectoryStream.Filter<Path> filter = file -> {
			return file.toString().endsWith(".mp4") || file.toString().endsWith(".MP4")
					|| file.toString().endsWith(".mov") || file.toString().endsWith(".MOV");
		};
		Path dirName = Paths.get("D:\\Project\\BE\\video");

		try (DirectoryStream<Path> stream = Files.newDirectoryStream(dirName, filter)) {
			stream.forEach(path -> {
				FFmpegFrameGrabber g = new FFmpegFrameGrabber(path.toString());
				try {
					g.start();
					Java2DFrameConverter paintConverter = new Java2DFrameConverter();
					BufferedImage difImage = paintConverter.getBufferedImage(g.grabImage());
					ImageIO.write(difImage, "png", new File(
							"D:\\Project\\BE\\video\\" + UUID.randomUUID().toString() + ".png"));
					g.stop();
					g.close();
				} catch (Exception e) {
					e.printStackTrace();
				}
			});
		} catch (IOException e1) {
			e1.printStackTrace();
		}

		String inputVideo = "D:\\Project\\BE\\video\\TWICE.mp4";
		String outputPreview = "D:\\Project\\BE\\video\\preview.mp4";
		try {
			generateVideoPreview(inputVideo, outputPreview, 10, 10);
			System.out.println("Preview video created successfully: " + outputPreview);
		} catch (Exception e) {
			System.err.println("Error creating video preview: " + e.getMessage());
		}
	}
	public static void generateVideoPreview(String inputPath, String outputPath, int startSecond, int duration) throws Exception {
		// Đọc video bằng FFmpegFrameGrabber
		FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(inputPath);
		grabber.start();

		// Chuyển tới thời điểm bắt đầu preview (microseconds)
		grabber.setTimestamp(startSecond * 1_000_000L);

		// Tạo file ghi video preview
		FFmpegFrameRecorder recorder = new FFmpegFrameRecorder(outputPath, grabber.getImageWidth(), grabber.getImageHeight(), grabber.getAudioChannels());

		recorder.setFormat("mp4");
		recorder.setFrameRate(grabber.getFrameRate());
		recorder.setVideoCodec(grabber.getVideoCodec());
		recorder.setAudioCodec(grabber.getAudioCodec());
		recorder.start();

		// Cắt video theo thời gian yêu cầu
		int framesToCapture = (int) (duration * grabber.getFrameRate());
		Frame frame;
		int frameCount = 0;

		while ((frame = grabber.grab()) != null && frameCount < framesToCapture) {
			recorder.record(frame);
			frameCount++;
		}

		// Dừng và giải phóng tài nguyên
		recorder.stop();
		recorder.release();
		grabber.stop();
		grabber.release();

		System.out.println("Preview video saved at: " + outputPath);
	}
}


## dependence
<dependency>
			<groupId>org.bytedeco</groupId>
			<artifactId>javacv-platform</artifactId>
			<version>1.5.9</version>
		</dependency>

  ## Có thể cài FFmpeg rồi sử dụng lệnh để convert
  ProcessBuilder builder = new ProcessBuilder(
    "ffmpeg", "-i", "input.mp4", "-ss", "00:00:05", "-t", "00:00:10", "-c:v", "libx264", "-preset", "fast", "preview.mp4"
);

builder.redirectErrorStream(true);
Process process = builder.start();
process.waitFor();
