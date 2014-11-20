---
layout: posts
title: "关于开发库的版本问题"
---

### 关于开发库的版本问题
ffmpeg抽取视频中motion vector的小结，
代码出自[这个网页](http://www.cs.ccu.edu.tw/~tsaicm/ExtractMV/myExtractMV.c)。使用[Zeranoe FFmpeg](http://ffmpeg.zeranoe.com/builds/)编译的库，也可以使用自己vs2010编译。

#### <font color="blue">建立工程</font>
1. 建立空白win32 console app
2. 新建C++文件，添加代码
3. $(ProjectDir)\ffmpegSDK\lib目录下创建目录ffmpegSDK，解压ffmpeg-2.0.1-win32-dev.7z中的include、lib到ffmpegSDK。
4. C/C++-->常规-->附加包含目录，添加$(ProjectDir)\ffmpegSDK\include;
5. 链接库-->常规-->附加库目录，添加$(ProjectDir)\ffmpegSDK\lib;
6. 链接库-->输入-->附加依赖项，<br>添加ws2_32.lib;avcodec.lib;avdevice.lib;avfilter.lib;avformat.lib;avutil.lib;postproc.lib;swresample.lib;swscale.lib;

#### <font color="blue">编译工程</font>
可见有很多错误，其中需要修改的是：

1. 注释掉
<pre class="prettyprint linenums">
#include <stdbool.h>
</xmp>
2. 添加extern "C"相关
3. FF_P_TYPE等改为AV_PICTURE_TYPE_P
4. av_open_input_file(&pFormatCtx, argv[1], NULL, 0, NULL)改为<br>avformat_open_input(&pFormatCtx, argv[1], NULL, NULL)
5. dump_format(pFormatCtx, 0, argv[1], false)改为<br>av_dump_format(pFormatCtx, 0, argv[1], false)
6. CODEC_TYPE_VIDEO改为AVMEDIA_TYPE_VIDEO
7. avcodec_decode_video(pCodecCtx, pFrame, &frameFinished, packet.data, packet.size)改为<br>avcodec_decode_video2(pCodecCtx, pFrame, &frameFinished, &packet)

#### <font color="blue">开始调试</font>
1. 生成成功之后，<font color="red">添加调试参数rtsp://192.168.1.158/H264,开始调试提示缺少dll文件，</font>需要下载相应dev版本的share版本并解压到工程的Debug目录。
2. 调试总是在<font color="red">if(avformat_open_input(&pFormatCtx, argv[1], NULL, NULL)!=0)位置中断，</font>由于版本升级需要把pFormatCtx初始化为NULL，<font color="red">好蛋疼的版本升级。</font>
3. 程序运行，但是控制台提示：<font color="blue">Using network protocols without global network initialization. Please use avform at_network_init(), this will become mandatory later.</font>需要添加avformat_network_init()到av_register_all();之后。
4. 总是在if (!USES_LIST(pict->mb_type[mb_index], direction))这个地方中断，网上找了几天的原因也没解决，主要的思路就是<br><font color="red">pCodecCtx->debug |= FF_DEBUG_MV | FF_DEBUG_MB_TYPE;</font><br><font color="red">pCodecCtx->debug_mv |= FF_DEBUG_VIS_MV_P_FOR | FF_DEBUG_VIS_MV_B_FOR | FF_DEBUG_VIS_MV_B_BACK;</font>但是同样的问题，不过确实出现了visual motion vector，但是还是中断在那个位置，<font color="blue">无奈之下，更换了ffmpeg-1.2-win32-dev这个版本之后，问题解决了。</font><font color="red">版本升级伤不起啊，开始还以为编译的时候出了问题，现在的看法是，新版本即使添加FF_DEBUG_MV，但avframe.mb_type还是不会被填充，遇到类似问题需要注意。</font>

最后附上修改过的代码
<pre class="prettyprint linenums">
extern "C"{
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libswscale/swscale.h>
}
#include <stdio.h>
#include <stdlib.h>
//#include <stdbool.h>

#define IS_INTERLACED(a) ((a)&MB_TYPE_INTERLACED)
#define IS_16X16(a)      ((a)&MB_TYPE_16x16)
#define IS_16X8(a)       ((a)&MB_TYPE_16x8)
#define IS_8X16(a)       ((a)&MB_TYPE_8x16)
#define IS_8X8(a)        ((a)&MB_TYPE_8x8)
#define USES_LIST(a, list) ((a) & ((MB_TYPE_P0L0|MB_TYPE_P1L0)<<(2*(list))))

static void SaveFrame(AVFrame *pFrame, int width, int height, int iFrame)
{
	FILE *pFile;
	char szFilename[32];
	int  y;

	// Open file
	sprintf(szFilename, "frame%d.ppm", iFrame);
	printf("Open File: frame%d.ppm\n", iFrame);
	pFile=fopen(szFilename, "wb");
	if(pFile==NULL)
		return;

	// Write header
	fprintf(pFile, "P6\n%d %d\n255\n", width, height);

	// Write pixel data
	for(y=0; y<height; y++)
		fwrite(pFrame->data[0]+y*pFrame->linesize[0], 1, width*3, pFile);

	// Close file
	fclose(pFile);
}

void SavePGM(AVFrame *pFrame, int width, int height, int iFrame)
{
	FILE *fp =0;
	int i,j,shift;
	uint8_t *y_factor;
	char szFilename[32];

	// Open file
	sprintf(szFilename, "%d.pgm", iFrame);
	fp = fopen(szFilename,"wb");
	if(fp) {
		// Write header
		fprintf(fp, "P5\n%d %d\n255\n", width, height);
		y_factor = pFrame->data[0];
		for(j = 0; j < height; j++) {
			fwrite(y_factor, width, 1, fp);
			y_factor += pFrame->linesize[0];
		}
		fclose(fp);
	}
}

void ffmpeg_dump_yuv(char *filename, AVPicture *pic, int width, int height)
{
	FILE *fp =0;
	int i,j,shift;
	uint8_t *yuv_factor;

	fp = fopen(filename,"wb");
	if(fp) {
		for(i = 0; i < 3; i++) {
			shift = (i == 0 ? 0:1);
			yuv_factor = pic->data[i];
			for(j = 0; j < (height>>shift); j++) {
				fwrite(yuv_factor,(width>>shift),1,fp);
				yuv_factor += pic->linesize[i];
			}
		}
		fclose(fp);
	}
}

/* Print motion vector for each macroblock in this frame.  If there is
* no motion vector in some macroblock, it prints a magic number NO_MV. */
void printMVMatrix(int index, AVFrame *pict, AVCodecContext *ctx)
{
	const int mb_width  = (ctx->width + 15) / 16;
	const int mb_height = (ctx->height + 15) / 16;
	const int mb_stride = mb_width + 1;
	const int mv_sample_log2 = 4 - pict->motion_subsample_log2;
	const int mv_stride = (mb_width << mv_sample_log2) + (ctx->codec_id == CODEC_ID_H264 ? 0 : 1);
	const int quarter_sample = (ctx->flags & CODEC_FLAG_QPEL) != 0;
	const int shift = 1 + quarter_sample;

	FILE * fpMVFile;
	char csMVFileName[100];
	sprintf(csMVFileName, ".\\%d.txt", index);
	fpMVFile = fopen(csMVFileName, "w");

	int mb_y, mb_x, type, i;
	for (mb_y = 0; mb_y < mb_height; mb_y++) {
		for (mb_x = 0; mb_x < mb_width; mb_x++) {
			const int mb_index = mb_x + mb_y * mb_stride;
			if (pict->motion_val) {
				for (type = 0; type < 3; type++) {
					int direction = 0;
					switch (type) {
					case 0:
						if (pict->pict_type != AV_PICTURE_TYPE_P)
							continue;
						direction = 0;
						break;
					case 1:
						if (pict->pict_type != AV_PICTURE_TYPE_B)
							continue;
						direction = 0;
						break;
					case 2:
						if (pict->pict_type != AV_PICTURE_TYPE_B)
							continue;
						direction = 1;
						break;
					}

					if (!USES_LIST(pict->mb_type[mb_index], direction)) {
#define NO_MV 10000
						if (IS_8X8(pict->mb_type[mb_index])) {
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
						}
						else if (IS_16X8(pict->mb_type[mb_index])) {
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
						}
						else if (IS_8X16(pict->mb_type[mb_index])) {
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
						}
						else {
							fprintf(fpMVFile, "%d\t%d\n", NO_MV, NO_MV);
						}
#undef NO_MV
						continue;
					}

					if (IS_8X8(pict->mb_type[mb_index])) {
						for (i = 0; i < 4; i++) {
							int xy = (mb_x*2 + (i&1) + (mb_y*2 + (i>>1))*mv_stride) << (mv_sample_log2-1);
							int dx = (pict->motion_val[direction][xy][0]>>shift);
							int dy = (pict->motion_val[direction][xy][1]>>shift);
							fprintf(fpMVFile, "%d\t%d\n", dx, dy);
						}
					}
					else if (IS_16X8(pict->mb_type[mb_index])) {
						for (i = 0; i < 2; i++) {
							int xy = (mb_x*2 + (mb_y*2 + i)*mv_stride) << (mv_sample_log2-1);
							int dx = (pict->motion_val[direction][xy][0]>>shift);
							int dy = (pict->motion_val[direction][xy][1]>>shift);

							if (IS_INTERLACED(pict->mb_type[mb_index]))
								dy *= 2;

							fprintf(fpMVFile, "%d\t%d\n", dx, dy);
						}
					}
					else if (IS_8X16(pict->mb_type[mb_index])) {
						for (i = 0; i < 2; i++) {
							int xy =  (mb_x*2 + i + mb_y*2*mv_stride) << (mv_sample_log2-1);
							int dx = (pict->motion_val[direction][xy][0]>>shift);
							int dy = (pict->motion_val[direction][xy][1]>>shift);

							if (IS_INTERLACED(pict->mb_type[mb_index]))
								dy *= 2;

							fprintf(fpMVFile, "%d\t%d\n", dx, dy);
						}
					}
					else {
						int xy = (mb_x + mb_y*mv_stride) << mv_sample_log2;
						int dx = (pict->motion_val[direction][xy][0]>>shift);
						int dy = (pict->motion_val[direction][xy][1]>>shift);
						fprintf(fpMVFile, "%d\t%d\n", dx, dy);
					}
				}
			}
		}
	}
	fclose(fpMVFile);
}

int main (int argc, const char * argv[])
{
	AVFormatContext *pFormatCtx = NULL;
	int             i, videoStream;
	AVCodecContext  *pCodecCtx;
	AVCodec         *pCodec;
	AVFrame         *pFrame; 
	AVFrame         *pFrameRGB;
	AVPacket        packet;
	int             frameFinished;
	int             numBytes;
	uint8_t         *buffer;

	// Register all formats and codecs
	av_register_all();
	avformat_network_init();

	// Open video file
	if(avformat_open_input(&pFormatCtx, argv[1], NULL, NULL)!=0)
	{
		printf("Couldn't open file\n");
		return -1; // Couldn't open file
	}

	// Retrieve stream information
	if(av_find_stream_info(pFormatCtx)<0)
	{
		printf("Couldn't find stream information\n");
		return -1; // Couldn't find stream information
	}

	// Dump information about file onto standard error
	av_dump_format(pFormatCtx, 0, argv[1], false);

	// Find the first video stream
	videoStream=-1;
	for(i=0; i<pFormatCtx->nb_streams; i++) {
		if(pFormatCtx->streams[i]->codec->codec_type==AVMEDIA_TYPE_VIDEO)
		{
			videoStream=i;
			break;
		}
	}
	if(videoStream==-1)
	{
		printf("Didn't find a video stream\n");
		return -1; // Didn't find a video stream
	}

	// Get a pointer to the codec context for the video stream
	pCodecCtx=pFormatCtx->streams[videoStream]->codec;

	// Find the decoder for the video stream
	pCodec=avcodec_find_decoder(pCodecCtx->codec_id);
	if(pCodec==NULL)
	{
		printf("Codec not found\n");
		return -1; // Codec not found
	}

	// Open codec
	if(avcodec_open(pCodecCtx, pCodec)<0)
	{
		printf("Could not open codec\n");
		return -1; // Could not open codec
	}

	// Hack to correct wrong frame rates that seem to be generated by some codecs
	if(pCodecCtx->time_base.num>1000 && pCodecCtx->time_base.den==1)
		pCodecCtx->time_base.den=1000;

	// Allocate video frame
	pFrame=avcodec_alloc_frame();

	// Allocate an AVFrame structure
	//pFrameRGB=avcodec_alloc_frame();
	//if(pFrameRGB==NULL)
	//    return -1;

	// Determine required buffer size and allocate buffer
	//numBytes=avpicture_get_size(PIX_FMT_RGB24, pCodecCtx->width,
	//    pCodecCtx->height);

	//buffer=malloc(numBytes);

	// Assign appropriate parts of buffer to image planes in pFrameRGB
	//avpicture_fill((AVPicture *)pFrameRGB, buffer, PIX_FMT_RGB24,
	//    pCodecCtx->width, pCodecCtx->height);

	// Read frames and save first five frames to disk
	i=0;
	printf("Extract MVs from video (mb_width=%d, mb_hight=%d):\n", (pCodecCtx->width + 15)/16, (pCodecCtx->height + 15)/16);
	while(av_read_frame(pFormatCtx, &packet)>=0)
	{
		// Is this a packet from the video stream?
		if(packet.stream_index==videoStream)
		{
			// Decode video frame
			avcodec_decode_video2(pCodecCtx, pFrame, &frameFinished, &packet);

			// Did we get a video frame?
			if(frameFinished)
			{
				if(i%50==0){
					printf("   ", i);
				}
				printMVMatrix(i++, pFrame, pCodecCtx);
				if (pFrame->pict_type == AV_PICTURE_TYPE_I){
					printf("I");
				}
				else if(pFrame->pict_type == AV_PICTURE_TYPE_P){
					printf("P");
					printMVMatrix(i, pFrame, pCodecCtx);
				}
				else if(pFrame->pict_type == AV_PICTURE_TYPE_B){
					printf("B");
				}
				else {
					printf(".");
				}
				i++;

				if(i%10==0) {
					printf(" ");
					if(i%50==0){
						printf("[ %d ]\n", i);
					}
				}

				/*
				static struct SwsContext *img_convert_ctx;

				// Convert the image into YUV format that SDL uses
				if(img_convert_ctx == NULL) {
				int w = pCodecCtx->width;
				int h = pCodecCtx->height;

				img_convert_ctx = sws_getContext(w, h, 
				pCodecCtx->pix_fmt, 
				w, h, PIX_FMT_RGB24, SWS_BICUBIC,
				NULL, NULL, NULL);
				if(img_convert_ctx == NULL) {
				fprintf(stderr, "Cannot initialize the conversion context!\n");
				exit(1);
				}
				}
				int ret = sws_scale(img_convert_ctx, pFrame->data, pFrame->linesize, 0, 
				pCodecCtx->height, pFrameRGB->data, pFrameRGB->linesize);

				// Save the frame to disk
				SaveFrame(pFrameRGB, pCodecCtx->width, pCodecCtx->height, i++);
				*/
			}
		}

		// Free the packet that was allocated by av_read_frame
		av_free_packet(&packet);
	}

	// Free the RGB image
	//free(buffer);
	//av_free(pFrameRGB);

	// Free the YUV frame
	av_free(pFrame);

	// Close the codec
	avcodec_close(pCodecCtx);

	// Close the video file
	av_close_input_file(pFormatCtx);

	return 0;
}
</xmp>