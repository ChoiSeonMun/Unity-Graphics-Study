# 6. UV를 이용하는 법을 배워 봅시다
- UV == XY == RG
- UV의 타입은 float2, 범위는 0.0 ~ 1.0 ==> 정규화 됨
- UV 배치는 엔진 or 툴마다 다름
    > 언리얼 엔진, DirectX는 좌측 상단이 원점
    > 유니티 엔진, OpenGL은 좌측 하단이 원점
- 시간과 관련된 변수
    > _Time / _SinTime / _CosTime / unity_DeltaTime

- Billboard

- 알파 채널은 게임에서 일반 오브젝트와 다르게 처리되며, 상당히 무거운 연산 중 하나다.

# 7. 버텍스 컬러를 이용해봅시다
- 버텍스에는 위치(Position), UV(Texcoord), 컬러(Color), 노멀(Normal), 탄젠트(Tangent) 등이 있다.
- 컬러의 기본값은 float4(1, 1, 1, 1)이다.
- 버텍스 컬러를 설정하려면 그래픽 툴에서 그리거나, 엔진의 자체 기능을 이용한다.
