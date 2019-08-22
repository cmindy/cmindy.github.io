---
layout: post
title:  "[iOS] 내 사진 관리 애플리케이션"
subtitle:   "부스트코스 iOS 프로젝트 4 내 사진 관리 애플리케이션"
categories: ios
tags: ios boostcourse retrospective
comments: true

---



나는 부스트코스 에이스 2019 1기로 활동중이다. 이 글은 그 중에서 iOS의 네번째 프로젝트인 기상정보 애플리케이션에 대한 회고이다.

아직 리뷰어가 지정이 안 돼서 답변은 받지 못해 내가 남긴 질문들을 정리했다😂



### Q. FetchResult vs Model

프로젝트에서 fetchResult를 활용해서 collection view에 정보들을 표시해주었다. 그런데 프로젝트를 진행하다보니까 이 방식에서 불편함을 느낀 부분이 몇 가지 있었다.

 그래서 `AlbumModel` 처럼 Model을 만들어서 앨범 이름이나, 에셋개수, [PHAsset] 등을 저장하고 이를 전반적으로 사용하는 것을 생각해봤었다.

 Model을 만들어서 Asset들을 저장하는 것과 fetchResult를 활용한 방법 중 어떤 것이 더 효율적인지 궁금했다.



### Q. 콜렉션 뷰 reload 시 requestImage()

콜렉션 뷰의 cellForItem에서 requestImage() 메서드를 호출해 받아온 이미지를 이미지 뷰에 세팅해준다.

이렇게 되면 ` collectionView.reloadData` 를 할 때마다 셀의 개수만큼 이미지를 요청하는 데, 이런 방식이 비효율적이라는 생각이 들었다. 

- 아직 리뷰는 못 받았지만 `NSCache` 등을 통한 이미지 캐싱을 통해 콜렉션뷰를 세팅할 수도 있을 것 같다.🧐



