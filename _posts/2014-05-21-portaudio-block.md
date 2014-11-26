---
layout: posts
title: "PortAudio阻塞模式"
---

# PortAudio阻塞模式
### 介绍
PortAudio源码包中例子paex_read_write_wire.c的注释。代码：
<xmp class="prettyprint linenums">
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "portaudio.h"

/* #define SAMPLE_RATE  (17932) // Test failure to open with this value. */
#define SAMPLE_RATE  (44100)    //采样率，每秒采样的次数，采样一次的数据称为一帧
#define FRAMES_PER_BUFFER (1024)//每次阻塞读写需要缓冲的帧数
#define NUM_CHANNELS    (2)		//单声道or双声道
#define NUM_SECONDS     (15)	//
/* #define DITHER_FLAG     (paDitherOff)  */
#define DITHER_FLAG     (0) /**/

/* @todo Underflow and overflow is disabled until we fix priming of blocking write. */
#define CHECK_OVERFLOW  (0)
#define CHECK_UNDERFLOW  (0)


/* Select sample format. *///采样值量化格式即数模转换，paFloat32, paInt32, paInt24, paInt16, paInt8 ...
#if 0
#define PA_SAMPLE_TYPE  paFloat32
#define SAMPLE_SIZE (4)
#define SAMPLE_SILENCE  (0.0f)
#define CLEAR(a) memset( (a), 0, FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%.8f"
#elif 0
#define PA_SAMPLE_TYPE  paInt16
#define SAMPLE_SIZE (2)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 1
#define PA_SAMPLE_TYPE  paInt24
#define SAMPLE_SIZE (3)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#elif 0
#define PA_SAMPLE_TYPE  paInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (0)
#define CLEAR(a) memset( (a), 0,  FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE )
#define PRINTF_S_FORMAT "%d"
#else
#define PA_SAMPLE_TYPE  paUInt8
#define SAMPLE_SIZE (1)
#define SAMPLE_SILENCE  (128)
#define CLEAR( a ) { \
	int i; \
	for( i=0; i<FRAMES_PER_BUFFER*NUM_CHANNELS; i++ ) \
	((unsigned char *)a)[i] = (SAMPLE_SILENCE); \
}
#define PRINTF_S_FORMAT "%d"
#endif


/*******************************************************************/
int main(void);
int main(void)
{
	PaStreamParameters inputParameters, outputParameters;
	PaStream *stream = NULL;
	PaError err;
	char *sampleBlock;
	int i;
	int numBytes;


	printf("patest_read_write_wire.c\n"); fflush(stdout);

	//每次阻塞读写缓冲区所占字节数=缓冲的帧数*声道数*量化大小
	numBytes = FRAMES_PER_BUFFER * NUM_CHANNELS * SAMPLE_SIZE ;
	sampleBlock = (char *) malloc( numBytes );					//开辟缓冲区
	if( sampleBlock == NULL )
	{
		printf("Could not allocate record array.\n");
		exit(1);
	}
	CLEAR( sampleBlock );										//清除缓冲区

	err = Pa_Initialize();
	if( err != paNoError ) goto error;

	inputParameters.device = Pa_GetDefaultInputDevice(); /* default input device */
	printf( "Input device # %d.\n", inputParameters.device );
	printf( "Input LL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultLowInputLatency );
	printf( "Input HL: %g s\n", Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency );
	inputParameters.channelCount = NUM_CHANNELS;
	inputParameters.sampleFormat = PA_SAMPLE_TYPE;
	inputParameters.suggestedLatency = Pa_GetDeviceInfo( inputParameters.device )->defaultHighInputLatency;
	inputParameters.hostApiSpecificStreamInfo = NULL;

	outputParameters.device = Pa_GetDefaultOutputDevice(); /* default output device */
	printf( "Output device # %d.\n", outputParameters.device );
	printf( "Output LL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultLowOutputLatency );
	printf( "Output HL: %g s\n", Pa_GetDeviceInfo( outputParameters.device )->defaultHighOutputLatency );
	outputParameters.channelCount = NUM_CHANNELS;
	outputParameters.sampleFormat = PA_SAMPLE_TYPE;
	outputParameters.suggestedLatency = Pa_GetDeviceInfo(outputParameters.device)->defaultHighOutputLatency;
	outputParameters.hostApiSpecificStreamInfo = NULL;

	/* -- setup -- */

	err = Pa_OpenStream(	//打开流
		&stream,
		&inputParameters,	//如果不需要输入流，则为NULL
		&outputParameters,
		SAMPLE_RATE,
		FRAMES_PER_BUFFER,
		paClipOff,      /* we won't output out of range samples so don't bother clipping them */
		NULL, /* no callback, use blocking API */
		NULL ); /* no callback, so no callback userData */
	if( err != paNoError ) goto error;

	err = Pa_StartStream( stream );//启动流，并不是通常意义的开始播放，需要write才能播放
	if( err != paNoError ) goto error;
	printf("Wire on. Will run %d seconds.\n", NUM_SECONDS); fflush(stdout);
	//(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER即总帧数/每次缓冲的帧数
	//也就是需要缓冲的次数
	for( i=0; i<(NUM_SECONDS*SAMPLE_RATE)/FRAMES_PER_BUFFER; ++i )
	{
		//阻塞写入FRAMES_PER_BUFFER帧，注意要与sampleBlock大小匹配
		err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_UNDERFLOW ) goto xrun;
		err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
		if( err && CHECK_OVERFLOW ) goto xrun;
	}
	err = Pa_StopStream( stream );//停止流
	if( err != paNoError ) goto error;
	//清除缓冲区
	CLEAR( sampleBlock );
	/*
	err = Pa_StartStream( stream );
	if( err != paNoError ) goto error;
	printf("Wire on. Interrupt to stop.\n"); fflush(stdout);

	while( 1 )
	{
	err = Pa_WriteStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	err = Pa_ReadStream( stream, sampleBlock, FRAMES_PER_BUFFER );
	if( err ) goto xrun;
	}
	err = Pa_StopStream( stream );
	if( err != paNoError ) goto error;

	Pa_CloseStream( stream );
	*/
	free( sampleBlock );//释放缓冲区
	//释放流
	Pa_Terminate();
	return 0;//正确返回

xrun:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	if( err & paInputOverflow )
		fprintf( stderr, "Input Overflow.\n" );
	if( err & paOutputUnderflow )
		fprintf( stderr, "Output Underflow.\n" );
	return -2;

error:
	if( stream ) {
		Pa_AbortStream( stream );
		Pa_CloseStream( stream );
	}
	free( sampleBlock );
	Pa_Terminate();
	fprintf( stderr, "An error occured while using the portaudio stream\n" );
	fprintf( stderr, "Error number: %d\n", err );
	fprintf( stderr, "Error message: %s\n", Pa_GetErrorText( err ) );
	return -1;
}
</xmp>