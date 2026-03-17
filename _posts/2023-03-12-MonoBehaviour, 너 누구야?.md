---
title: "[Unity] MonoBehaviour, 너 누구야?"
date: 2023-03-12 13:17 +0900
description: "Unity MonoBehaviour 클래스의 역할과 동작 원리를 알아봅니다."
categories:
  - Game
  - Unity
tags:
  - tutorial
---
  이번 "레트로의 유니티" 2부에서는 전반적으로 유니티에서 사용될 C# 문법에 대해 다루었다. 이전 글에서는 C#스크립트를 생성하면 나오는 기초 함수에 대해 다루었다면, 이번엔 스크립트의 기본 클래스에 계속 상속되어 있는 MonoBehaviour에 대해 다뤄볼 까 한다. 얘는 대체 뭐길래 처음부터 이렇게 상속을 받고 시작하나 궁금했다. 새로 스크립트를 생성할때 기본 이름도 NewBehaviour이고, 뭔가 있을거 같고 궁금해서 이렇게 찾아보게 되었다.

  

## **브로드캐스팅**

 유니티의 모든 컴포넌트는 MonoBehaviour를 상속받는데, MonoBehaviour는 컴포넌트에 필요한 기본 기능을 제공한다. 그렇기에 MonoBehaviour를 상속받은 컴포넌트는 유니티의 제어를 받게 되고, 그렇기에 브로드캐스팅에 관여함을 알 수 있다.

  

## **라이프사이클**

유니티의 공식 메뉴얼을 참고하면 여러 유니티의 라이프사이클에 관련된 함수들을 있음을 알 수 있고, MonoBehaviour은 라이프사이클에 관여됨을 알 수 있었다.

![](assets/img/post/2023/03_12_unity_lifecycle.png)
_유니티 라이프사이클_
{: width="400px"}


  

## **총평**

 이번에 MonoBehaviour이 지속적으로 등장하기도 하고, 기본적으로 상속받는것이 궁금해 찾아보게 됐다. 어느정도 MonoBehaviour은 유니티의 기본적인 기능들에 대해서 구현을 해둔 클래스이고, 그렇기에 모든 곳에서 상속을 받고 있구나 알게 되었다.

  

 좀더 잘 정리된 자료들이 많았어서 아래에 링크 달고 나중에 나도 또 읽어보고 해야겠다.

  

## **참고자료**

> 유니티 공식 문서 : [https://docs.unity3d.com/kr/530/ScriptReference/MonoBehaviour.html](https://docs.unity3d.com/kr/530/ScriptReference/MonoBehaviour.html)
> 블로그 1 : [https://skuld2000.tistory.com/25](https://skuld2000.tistory.com/25)
> 블로그 2 : [https://luv-n-interest.tistory.com/1299](https://luv-n-interest.tistory.com/1299)