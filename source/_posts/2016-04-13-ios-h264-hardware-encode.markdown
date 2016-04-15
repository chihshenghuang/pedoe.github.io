---
layout: post
title: "iOS H264 Hardware Encode"
date: 2016-04-13 16:33:42 +0800
comments: true
categories: 
-CSS3
-Sass
-Media Queries
---

##How to encode h264 file on iPhone by hardware encoding

##前言

最近工作上需要在iPhone上處理影音方面的codec, 花了不少時間去study跟找資料, 所以想把過程記錄下來, 怕以後自己忘掉可以很快回想起來。也希望可以幫到有需要的人,如果有許多錯誤還請多包涵, 畢竟小弟我只是個寫程式的新手。

要做到視訊影音的傳輸, 大概可以分成四個部分

1. 影像encode
2. 影像decode
3. 聲音播放
4. 聲音錄製

這篇就先記錄影像encode的部分, 為了達到encode出 h264 file的目的, 這邊的例子為用iPhone錄製影像, 並將影像encode成h264 file再輸出, 大概可以分成錄製影像流程跟硬體encode流程這兩個部分做說明。

##錄製影像

這部分網路上資料很多, google就有很多答案, 主要的流程可以分為

1. 選擇輸入來源(相機, 麥克風, 耳機等等)
2. 選擇前鏡頭後鏡頭
3. 建立data output buffer
4. 建立AVCaptrueSession
5. 設定session相關參數

這裡主要提一下AVCaptureSession, 根據Apple官方文件的說明, AVCaptureSession用來控制聲音或影像的輸入與輸出。就是先建立AVCaptureSession後, 加入輸入源例如AVCaptureDevieInput以及輸出源例如AVCaptureMovieFileOutput, 參考Apple官方的sample code如下:

```
AVCaptureSession *captureSession = [[AVCaptureSession alloc] init];
AVCaptureDevice *audioCaptureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeAudio];
NSError *error = nil;
AVCaptureDeviceInput *audioInput = [AVCaptureDeviceInput deviceInputWithDevice:audioCaptureDevice error:&error];
if (audioInput) {
	    [captureSession addInput:audioInput];
}
else {
	    // Handle the failure.
}
```
其他設定相機錄製等等就到後面直接看程式碼說明會比較清楚。

##Hardware Encode 流程

在以前蘋果是沒有開放hardware codec給開發者使用的, 到了iOS8.0後才推出了VideoToolbox這個framework來讓開發者使用, 藉由硬體將需要大量運算的影像編解碼處理掉, 可以省下不少運算資源留給其他功能。為了使用硬體進行encode, 我們可以先參考蘋果在WWDC14中session513(Direct Access to Video Encoding and Decoding)中對encode流程的示意圖如下:

   ![compress flow](/images/iOS_H264_HardwareEncode/CompressFlow.png)

這張圖要描述一開始iPhone鏡頭(最左邊)開啟後開始拍攝, 資料流開始讀入, 並假設現在在拍那隻貓, 貓的影像就會一張一張存到Buffer中(此時是為壓縮的原始檔)。

再來中間是AVAssetWriter, 這部分將未壓縮的frame丟到Video Encoder中轉成H.246的格式再輸出成file。這個部分可以再拆更細來看如下圖:
	
   ![compression session](/images/iOS_H264_HardwareEncode/CompressionSession.png)

為了使用video encoder, 我們必須創建VTCompressionSession這個物件, 其中VTCompressionSession這個物件必須給定部分條件或值才能成功建立, 條件如下	

1. Dimenssions for compressed output
2. Format for compression
3. PixelBufferAttributes describing source buffer requirements (optional)
4. A callback for VTCompresssionSession output

第一個就是你必須告知encoder你想要壓縮的frame大小, 大小單位為pixel; 第二個是則是你想壓縮成的格式, 在Apple官方提供的framework中有幾種格式可以選, 不過這裡格式就用H.264;
第三個為看你要不要建立buffer pool來存放你的source frame, 這部分我沒有用到就不是非常清楚; 第四個為必須給一個callback讓encoder處理完後呼叫, callback的機制可以參考下圖

   ![compression callback](/images/iOS_H264_HardwareEncode/CompressionCallback.png)

可以看出來我們必須要將壓縮後存有h.264 frame的CMSampleBuffer傳入callback, 並在callback中進行h.264 frame的解析處理(關於sps, pps ,i-frame etc.), 若在中間錯誤callback則會有
error code等相關處理。

創建完VTCompressionSession這個物件後, 可以對該encoder給定一些描述, 例如壓縮的bitrate, 畫素等等如下圖:

   ![compression configuring](/images/iOS_H264_HardwareEncode/CompressionConfiguring.png)

整個流程可以總結如下

1. 準備一個裝有未壓縮frame的Buffer
2. 建立encoder(前面提到的VTCompressionSession物件)
3. 設定encoder相關參數, 特性
4. 開始錄影後把得到的CVPixelBuffer送進encoder內, 再把encode出來的frame放入CMSampleBuffer做後續的處理(看要存成檔案還是透過網路傳出去等等)


##實作錄製影像並encode成h264檔案

###開啟錄影等功能

首先開啟一個新的專案

   ![create project](/images/iOS_H264_HardwareEncode/CreateProject.png)

建立一個Button跟UIView, Button用來開始錄製和結束錄製, UIView則用來呈現出鏡頭拍攝到的即時影像。

   ![set view and button on storyboard](/images/iOS_H264_HardwareEncode/UIView.png)

接下來加入VideoToolBox的framework和把UIView跟ViewController做連結

   ![videoToolBox](/images/iOS_H264_HardwareEncode/myView_VideoToolBox.png)

接著先處理相機相關的部分, 根據上面介紹的流程, 我們先建立物件如下, 並且使class符合<AVCaptureVideoDataOutputSampleBufferDelegate> 這個協定的規範
```
@interface ViewController ()<AVCaptureVideoDataOutputSampleBufferDelegate>
{
	AVCaptureSession *captureSession;          //根據前面說明captureSession負責處理資料的輸入輸出
	AVCaptureVideoPreviewLayer *previewLayer;  //到時候將鏡頭拍攝到的影像在此view上作呈現
	AVCaptureConnection *connection;           //負責資料流的開關與建立captureSession後和capture到的output, input資料做連結
	bool isStart;                              //開啟和關閉影像錄製, 初始值給定true
}
@end
```
再來建立Button觸發後的行為

```
- (IBAction)actionStartStop:(id)sender {
	if (isStart) {
		[self startCamera];
		isStart = false;
		[_startButton setTitle:@"Stop" forState:UIControlStateNormal];
	}
	else{
		isStart = true;
		[self stopCamera];
		[_startButton setTitle:@"Start" forState:UIControlStateNormal];
	 }
}

- (void)startCamera
{
	    
}

- (void)stopCamera
{
	    
}
```

接下來處理button觸發後startCamera的行為, 首先建立輸入來源, 在這邊我們選擇前鏡頭, 方便自拍

```
NSError *deviceError;
AVCaptureDeviceInput *inputDevice;
for (AVCaptureDevice *device in [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo]) {
	if ([device position] == AVCaptureDevicePositionFront) {
		inputDevice = [AVCaptureDeviceInput deviceInputWithDevice:device error:&deviceError];
	}
}
```

處理輸出源, 這邊使用了AVCaptureVideoDataOutput這個類別, 因為它提供我們去access得到的未壓縮frame, 可以呼叫 captureOutput:didOutputSampleBuffer:fromConnection:
這個delegate method來對frame做後續處理製作成h264的frame。除了選定輸出源, 我們還可以對輸出源做參數上的設定, iOS對kCVPixelBufferPixelFormatTypeKey支援三種format分別是

	kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange 
	kCVPixelFormatType_420YpCbCr8BiPlanarFullRange
	kCVPixelFormatType_32BGRA.

詳細差異我就不是很清楚了

```
AVCaptureVideoDataOutput *outputDevice = [[AVCaptureVideoDataOutput alloc]init];
NSString *key = (NSString *)kCVPixelBufferPixelFormatTypeKey;	    
NSNumber *val = [NSNumber numberWithUnsignedInt:kCVPixelFormatType_420YpCbCr8BiPlanarFullRange];
NSDictionary *videoSettings = [NSDictionary dictionaryWithObject:val forKey:key];
outputDevice.videoSettings = videoSettings;
[outputDevice setSampleBufferDelegate:self queue:dispatch_get_main_queue()];
```	

allocate AVCaptureSession, 接著加入output與input, 再呼叫begingConfiguring為後續調整output做準備。 
這邊我們設定畫素為640x480, 並將connection設為連接output與video, 最後commit我們的配置。 

``` 
captureSession = [[AVCaptureSession alloc]init];
[captureSession addInput:inputDevice];
[captureSession addOutput:outputDevice];				    
[captureSession beginConfiguration];
captureSession.sessionPrest = AVCaptureSessionPreset640x480;
connection = [outputDevice connectionWithMediaType:AVMediaTypeVideo];
[connection commitConfiguration];
```

最後把擷取到的影像放到previewLayer上做呈現, 並startRunning!

``` 
previewLayer = [[AVCaptureVideoPreviewLayer alloc]initWithSession:captureSession];
previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
[self.myView.layer addSublayer:previewLayer];
previewLayer.frame = self.myView.bounds;
[captureSession startRunning];
```

停止的部分比較簡單, 呼叫stopRunning即可
```
- (void)stopCamera
{	
	[captureSession stopRunning];
}
```
到這邊應該就可以試看看簡單的攝影功能了, 按下button相機鏡頭拍攝到的影像就會呈現在preView上。 

###Encoder建立

先新增一個名為VideoEncode的class, 再VideoEncode.h檔中, 我們建立兩個method與新增一個delegate 的portocol協定
initEncode:width:height是用來初始化一個encoder, 並給定要encode的影像畫素大小; encode:則是把未壓縮的frame做encode, 需要傳入一裝有未壓縮frame的sampleBuffer
這邊則創建一個新的portocol協定取名為H264HwEncoderDelegate, 裡面含有兩個實作方法, 分別是取得sps, pps frame還有keyframe的方法。接下來只要在ViewController中實作該方法就可以得到
影像再存入檔案中了。 
```
#import <Foundation/Foundation.h>
#import <AVFoundation/AVFoundation.h>
#import <VideoToolbox/VideoToolbox.h>

@protocol H264HwEncoderDelegate <NSObject>

- (void)gotSpsPps:(NSData*)sps pps:(NSData*)pps;
- (void)gotEncodedData:(NSData*)data isKeyFrame:(BOOL)isKeyFrame;

@end

@interface VideoEncode : NSObject
- (void) initEncode:(int)width  height:(int)height;
- (void) encode:(CMSampleBufferRef )sampleBuffer;

@property (weak, nonatomic) id<H264HwEncoderDelegate> delegate;

@end
```

接下來宣告相關變數, VTCompressionSessionRef就是前面提到的VideoToolbox提供的壓縮session物件, encodeQueue則是要將encode frame丟到queue裡面照順序處理, 得到的file時間順序才會對。
CMFormatDescriptionRef是用來設定壓縮的format; 
```
@implementation VideoEncode 
{
	NSString *error;	 
	VTCompressionSessionRef *encodeSession;
	dispatch_queue_t encodeQueue;
	CMFormatDescriptionRef format;
	BOOL initialized;
	int frameCount;
	NSData *sps;
	NSData *pps;
}
```

進行初始化
```
- (id)init
{
	self = [super init];
	if(self) {			        
		[self initVariables];
	}
	return self;
}

- (void)initVariables
{
	encodeSession = nil;
	initialized = true;
	encodeQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	frameCount = 0;
	sps = NULL;
	pps = NULL;
}
```

初始化encoder內容, 先將初始化的過程丟到sync Queue內保護起來照順序做。接著建立Compression Session, VTCompressionSessionCreate這個API有許多東西要給入, 例如畫素大小, 
callbalck(didCompressH264), 還有最一開始宣告的encodeSession位置等等。而VTCompressionSessionCreate這個API會回傳一個error code, 正常的話會傳回0, 方便我們檢驗。
VTSessionSetProperty有很多參數可以設定, 這裏設定了realtime的encode, I frame到下一張I frame的間隔, 以及選定compression bitstream的profile level。
最後告知encoder準備好了。

```
- (void)initEncode:(int)width height:(int)height
{
	dispatch_sync(encodeQueue, ^{
    	// Create the compression session
		OSStatus status = VTCompressionSessionCreate(NULL, width, height, kCMVideoCodecType_H264, NULL, NULL, NULL, didCompressH264, (__bridge void *)(self), &encodeSession);
		NSLog(@"H264: VTCompressionSessionCreate %d", (int)status);
															        
		if (status != 0) {
			NSLog(@"H264: Unable to create H264 session");
			error = @"H264: Unable to create H264 session";
		
			return;
		}
																																		       
		// Set the properties
		VTSessionSetProperty(encodeSession, kVTCompressionPropertyKey_RealTime, kCFBooleanTrue);
		VTSessionSetProperty(encodeSession, kVTCompressionPropertyKey_MaxKeyFrameInterval, (__bridge CFNumberRef)@(10.0)); // change the frame number between 2 I frame
		VTSessionSetProperty(encodeSession, kVTCompressionPropertyKey_ProfileLevel, kVTProfileLevel_H264_Main_AutoLevel);
																																												
		// Tell the encoder to start encoding
		vVTCompressionSessionPrepareToEncodeFrames(encodeSession);
	});
}
```

處理encode frame部分, 前面init時不丟到queue裡面還行, 這裏注意一定要丟到sync queue裡, 確保frame的順序正確。
使用CMSampleBufferGetImageBuffer來取得buffer內的影像資料, 存到imageBuffer內; VTCompressionSessionEncodeFrame, 這個function就是負責壓縮frame的function, 所以有一些參數必須設置
以及傳入。一些重要一定得傳入的有先前創建的encodeSession, 還有存有未壓縮frame的imageBuffer, 最後則是當error發生時要進行的處理。

```
- (void)encode:(CMSampleBufferRef)sampleBuffer
{
	dispatch_sync(encodeQueue, ^{
		frameCount++;
		// Get the CV Image buffer
		CVImageBufferRef imageBuffer = (CVImageBufferRef)CMSampleBufferGetImageBuffer(sampleBuffer);
		// Create properties
		CMTime presentationTimeStamp = CMTimeMake(frameCount, 1000);
		//CMTime duration = CMTimeMake(1, DURATION);
		VTEncodeInfoFlags flags;
		// Pass it to the encoder
		OSStatus statusCode = VTCompressionSessionEncodeFrame(encodeSession,
				imageBuffer,
				presentationTimeStamp,
				kCMTimeInvalid,
				NULL, NULL, &flags);
		// Check for error
		if (statusCode != noErr) {
			NSLog(@"H264: VTCompressionSessionEncodeFrame failed with %d", (int)statusCode);
			error = @"H264: VTCompressionSessionEncodeFrame failed ";
			
			// End the session
			VTCompressionSessionInvalidate(encodeSession);
			CFRelease(encodeSession);
			encodeSession = NULL;
			error = NULL;
		
			return;
		}
			
			NSLog(@"H264: VTCompressionSessionEncodeFrame Success");
	});
}
```

關於callback比較特別, 進去看VTCompressionCreate這個API裡面的Callback是一個C語言的function pointer type, 所以要做一些轉換。要宣告該callback函數讓其他method能夠呼叫, 
再實作該callbackvoid。callback要處理的事情有點多就分段記錄下來, 首先是檢查呼叫callback時傳入的狀態與資料是否正確以及是否有key frame等等, 再來是因為在C語言的寫法下我們沒辦法像[self function]這樣呼叫, C語言看不懂self, 所以在呼叫callback時要把整個class傳入再做casting的轉換。 

```
void didCompressH264(void *outputCallbackRefCon, void *sourceFrameRefCon, OSStatus status, VTEncodeInfoFlags infoFlags,CMSampleBufferRef sampleBuffer );

void didCompressH264(void *outputCallbackRefCon, void *sourceFrameRefCon, OSStatus status, VTEncodeInfoFlags infoFlags,CMSampleBufferRef sampleBuffer )
{
	 if (status != 0) {
	 	assert(status == 0);
		return;	
	}
	
	if (!CMSampleBufferDataIsReady(sampleBuffer)) {
		NSLog(@"didCompressH264 data is not ready");
	}
	
	VideoEncode *THIS = (__bridge VideoEncode*)outputCallbackRefCon;
																	     
	//Check if we have got a key frame first
	bool keyframe = !CFDictionaryContainsKey((CFArrayGetValueAtIndex(CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true), 0)), kCMSampleAttachmentKey_NotSync);

```
若成功得到key frame後, 依照h264的格式先後得到sps, pps, i-frame。根據官方提供的資料, 在錄製影像時未壓縮的檔案是以MP4的格式header用AVCC呈現。可以看下圖會比比較清楚的了解, 
可以看到下圖中MP4的sps, pps可以直接利用CMVideoFormatDescriptionGetH264ParameterSetAtIndex這個method來取得。

   ![H264 sps pps format](/images/iOS_H264_HardwareEncode/H264_sps_pps.png)


知道如何取得sps, pps後先宣告frame的資料大小以及存放frame raw data的變數, 接下來呼叫CMVideoFormatDescriptionGetH264ParameterSetAtIndex時
將宣告的變數位址傳入。得到sps, pps frame後就可以將資料丟到有實作前面定義portocol協定的class中, 在這邊是ViewController, 因此ViewController那邊就可以得到sps, pps frame的資料。
```
	if (keyframe) {
		CMFormatDescriptionRef format = CMSampleBufferGetFormatDescription(sampleBuffer);
		size_t sparameterSetSize, sparameterSetCount;
		const uint8_t *sparameterSet;
		OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format,0,&sparameterSet,&sparameterSetSize,&sparameterSetCount, 0);
																											     
		if (statusCode == noErr) {
			assert(status == 0);
			// Found sps and now check the pps
			size_t pparameterSetSize, pparameterSetCount;
			const uint8_t *pparameterSet;
			OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format,1,&pparameterSet,&pparameterSetSize,&pparameterSetCount, 0);
																															
			if (statusCode == noErr) {
				assert(status == 0);
				// Found pps
				THIS->sps = [NSData dataWithBytes:sparameterSet length:sparameterSetSize];
				THIS->pps = [NSData dataWithBytes:pparameterSet length:pparameterSetSize];
			
				if (THIS->_delegate) {
					[THIS->_delegate gotSpsPps:THIS->sps pps:THIS->pps];
				}
			}
		}
	}
```

處理完sps, pps frame後進行i-frame的處理。i-frame跟sps, pps frame比起來就需要做一些轉換, MP4的前四個byte代表該筆資料的長度, 第五個byte開始才是影像資料。
所以我們需要另外處理一下, 而且前四個byte是以big-endian呈現資料長度, 需要轉成low-endian。剩下的就是不斷的把位置移到下一個NALU開頭, 找到之後再把資料依序處理。
這部分可以參考下圖

   ![h264 i frame](/images/iOS_H264_HardwareEncode/H264_iFrame.png)


因為i-frame不確定長度會有多長, 有可能會比較大的長度才得到完整的i-frame, 這裏用CMBlockBufferGetDataPointer去取得資料。一樣把要存放資料與資料的大小長度等變數位址傳入。
```
	CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer);
	size_t length, totalLength;
	char *dataPointer;
	OSStatus statusCodeRet = CMBlockBufferGetDataPointer(dataBuffer, 0, &length, &totalLength, &dataPointer);
	if (statusCodeRet == noErr) {
		assert(statusCodeRet == 0);
		size_t bufferOffset = 0;
		static const int AVCCHeaderLength = 4;
		while (bufferOffset < totalLength - AVCCHeaderLength) {
			// Read the NAL unit length
			uint32_t NALUnitLength = 0;
			memcpy(&NALUnitLength, dataPointer + bufferOffset, AVCCHeaderLength);
		
			// Convert the length value from Big-endian to Little-endian
			// After converting the length value, the start 4 byte will present the NALUnitLength
		
			NALUnitLength = CFSwapInt32BigToHost(NALUnitLength);
			NSData *data = [[NSData alloc]initWithBytes:(dataPointer + bufferOffset + AVCCHeaderLength) length:NALUnitLength];
																																									            
			if (THIS->_delegate) {
				[THIS->_delegate gotEncodedData:data isKeyFrame:keyframe];
			}
		
			// Move to the next NAL unit in the block buffer	
			bufferOffset += AVCCHeaderLength + NALUnitLength;
			}	
	}
```

回到ViewController來看, 將宣告的變數補齊以及實作delegate
```
@interface ViewController ()<AVCaptureVideoDataOutputSampleBufferDelegate, H264HwEncoderDelegate>
{
	AVCaptureSession *captureSession;
	AVCaptureVideoPreviewLayer *previewLayer;
	AVCaptureConnection *connection;
	bool isStart;
	VideoEncode *videoEncode;
	NSFileHandle *fileHandle;
	NSString *h264File;
}
@property (weak, nonatomic) IBOutlet UIButton *startButton;
@end
```

接著透過delegate取得資料後寫入檔案, 在前面有用到AVCaptureVideoDataOutput並實作該協定, 所以加上該協定下的delegate method。
這個method會在有資料進到buffer內時被呼叫, 我們再將buffer傳給繼承VideoEncode的物件videoEncode去做影像的encode。 

```
-(void) captureOutput:(AVCaptureOutput*)captureOutput didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection*)connection
{
	[videoEncode encode:sampleBuffer];
}
```


接著在ViewController內實作H264HwEncoderDelegate的方法來取得sps, pps, i-frame, 記得要在寫入file前將frame的資料加上h264的header byte(0x00, 0x00, 0x00, 0x01)
因為MP4 AVCC下沒有h264的header, 要自己加上去。
```
- (void)gotSpsPps:(NSData*)sps pps:(NSData*)pps
{
	NSLog(@"gotSpsPps %d %d", (int)[sps length], (int)[pps length]);
	const char bytes[] = "\x00\x00\x00\x01";
	size_t length = (sizeof bytes) - 1; //string literals have implicit trailing '\0'
	NSData *ByteHeader = [NSData dataWithBytes:bytes length:length];
	[fileHandle writeData:ByteHeader];
	[fileHandle writeData:sps];
	[fileHandle writeData:ByteHeader];
	[fileHandle writeData:pps];									    
}

- (void)gotEncodedData:(NSData*)data isKeyFrame:(BOOL)isKeyFrame
{
	NSLog(@"gotEncodedData %d", (int)[data length]);
	if (fileHandle != NULL)
	{
		const char bytes[] = "\x00\x00\x00\x01";
		size_t length = (sizeof bytes) - 1; //string literals have implicit trailing '\0'
		NSData *ByteHeader = [NSData dataWithBytes:bytes length:length];
		[fileHandle writeData:ByteHeader];
		[fileHandle writeData:data];	
	}
}
```

到這邊應該就差不多完成了, 可以簡單的錄製影像成h264 file, 再找個可以播放h264的播放器驗證看看。我是用VLC或是用自己寫的另一個專案做h264 decoder, 
這部分找機會會在記錄一下, 畢竟打一篇網誌真的要好久好久...

可能有部分小細節沒有提到, 所以我將這部分的代碼放在我的GitHub上: https://github.com/pedoe/iOS_h264_hardware_encode

有問題的地方還請告知~
