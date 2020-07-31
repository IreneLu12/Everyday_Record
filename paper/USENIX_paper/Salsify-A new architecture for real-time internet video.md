# Salsify: A new architecture for real-time internet video

## Introduction
- 当前系统不能快速应对网络变化，最终会导致网络拥塞，造成停顿和故障
- Salsify是一个用于实时互联网视频的架构
- Salsify结合了**视频感知的传输层协议**和**功能性的视频编码器**，可以快速响应网络状况
- 相比FaceTime, Hangouts, Skype和WebRTC，Salsify取得了更高的质量(SSIM)和更低延迟

## Architecture
- 当前的视频传输系统包括两个相对独立的模块：视频编码器和传输层协议。两者分别有一个单独的控制循环，传输层协议会给编码器给定一个**目标码率**，而编码器将压缩的视频帧送入传输层。这样的问题是编码器控制的是平均码率，比如在编码I帧的时候码率超过目标码率，就会在编码P帧的时候降低编码速率。因此如果编码器产生了一个高码率的视频帧，传输层协议依旧会将其发送到网络，导致拥塞。
- 传统设计的问题是编码器只能保证输出码率的平均值为目标码率，因此单独的I帧依然可能导致网络拥塞。这导致传统系统设计对变化的网络状况的反应很慢
- Salsify的传输层协议受到*packet pair*和*Sprout-EWMA*的启发，视频编码格式为*VP8*，功能性编码器已经在*NSDI'17*上发表
- 传输层协议估计不引起过量延迟的的下一帧大小，而不是比特率。
- 功能编码器可以探索不同的路径，对于每一帧，编码器传输的数据有三种选择：传输一个高质量的帧；传输一个低质量的帧；跳过这一帧。根据计算的目标帧大小做出决策。

## Testbed
- 系统包括可重现的输入视频，可重现的网络轨迹
- 目标QoE：每一帧的质量和每一帧的延迟，通过在每一帧插入二维码进行匹配，并测量指标

## Evaluation
- 在LTE网络和T-Mobile UMTS轨迹下，传统编码器+Salsify传输协议达到了低质量低延迟，完全的Salsify达到了超越所有对比方案的质量和低延迟
- 但是在WIFI下，它的延迟比WebRTC低，但是它的质量更低
- 当网络急速恶化再好转时，它的带宽反应速度快于WebRTC，可以很好地调整吞吐量适应当前带宽

## QA
- 在移动端，视频编码器一般在ASIC，进行改变的成本很高
	- 如果不能在硬件部署该编码器，很难在移动设备上使用。本论文提倡硬件厂商在下一代编码器中纳入此界面，目前只有该界面的软件版本。
- 在Salsify中，编码器需要同时产生两个版本的视频帧，明显提高了计算成本
	- 编码器给一个帧编码了两个版本，会增加2X的计算成本。作者认为每秒生产60帧的720P视频是不难的，所以依旧可以保持很高的帧率
- 如果编码器已经产生了I帧，它是否可以选择不同质量的P帧，还是说要等到下一个I帧？
- 为什么LTE环境比WIFI好很多？
	- 因为Salsify是为变化网络环境设计的，在比较稳定的网络环境下表现没有WebRTC好
