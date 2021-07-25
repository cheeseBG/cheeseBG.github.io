---
date: 2021-07-25
title: "Nexmon CSI extractor: Raspberry Pi 4 설정"
categories: Network
tags: CSI
# 목차
toc: true  
toc_sticky: true 
---
CSI 데이터 수집을 위해 Nexmon에서 제공하는 CSI extrator 환경을 Raspberry Pi 4에 구축했다.

해당 깃허브 사이트([https://github.com/seemoo-lab/nexmon_csi](https://github.com/seemoo-lab/nexmon_csi))에서 README를 보고 그대로 진행했으나 CSI 데이터가 추출되지 않는 문제가 있어 다양한 시행착오 끝에 성공한 해결방법을 정리하려고 한다.

### 1. Firmware patch
가장 먼저 README의 'Getting Started'의 'bcm43455c0' 파트를 진행하게되는데 이 과정에서 에러가 발생한다면 kernel 버전을 확인해봐야 한다.

*kernel 버전을 반드시 4.19 또는 5.4로 맞추고 진행해야한다!*

모든 과정을 마치고 firmware patch에 성공하더라도 재부팅하는 경우 firmware patch 과정을(Getting Started'bcm43455c0' 8번~12번) 다시 진행해야하는 번거로움이 있다.

이 부분에 대해서는 추후에 해결방법을 찾아 포스팅 할 예정이다.

### 2. Usage
내 경우 Firmware patch 이후 README Usage를 따라 진행했을때, tcpdump로 CSI가 담긴 packet 추출을 시도하면 packet이 하나도 추출되지않았다. 

알아본 결과 CSI parameter를 만드는 과정에서 channel, bandwith 등 몇개의 parameter 설정에 문제가 있었다.

따라서 같은 문제(packet이 추출되지 않는)가 발생한다면 아래 사항들을 따라해보길 바란다.

- AP를 2.4MHz bandwidth 36 channel로 세팅 (36 channel 보다 높은 채널들은 아직 문제가 있다고한다)
- 아래에서 `157/80`을 `36/20`으로 변경(그래도 안될경우 `nexutil -k`를 통해 channel, bandwidth 값 확인후 입력), `-b 0x88` 제거
 
  ```
    makecsiparams -c 157/80 -C 1 -N 1 -m 00:11:22:33:44:55 -b 0x88
  ```
  
  `-b`의 경우 설정한 byte와 매칭되는 CSI만 추출한다고 하는데 Issue를 확인해보면 사용할 경우 다들 오류가 난다고 한다(내가 확인해봤을때는)

### 3. Another problems
위 방법들로 CSI 추출에 성공했지만 일정한 간격으로 정해진 수의 CSI packet을 받지 못하고 있다.

또한 tcpdump로 packet을 받다보면 중간에 packet 수가 늘어나지 않게되는데 이때 `nexutil -k`를 입력하면 결과값이 `chanspec: 0x6863, 85/160`가 나온다.
이 경우 extractor가 제대로 작동하지 않아 CSI를 더이상 추출할 수 없게되는데 정확한 이유는 아직 찾지 못했다.(다른 사용자들도 같은 문제가 발생한다고 한다)

이 문제들은 해결법을 찾게된다면 포스트를 업데이트 할 예정이다.

  
참고로 Nexmon CSI collector가 CSI를 추출하는 방식은 아래 그림과 같다.
<br/> 
<img src = "https://user-images.githubusercontent.com/51084152/126897905-cf78e73c-5c64-4aca-a599-30d77d27181f.png" width="300px">
<br/>
