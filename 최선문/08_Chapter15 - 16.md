# 15. Cubemap Reflection
- Cubemap : 어떤 오브젝트의 주변 환경에 대한 반사를 나타내는 6개의 사각형 텍스처 컬렉션이다. 각각의 면은 월드 좌표와 대응된다.
![큐브맵을 펼쳐놓은 사진](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Cube_map.svg/1920px-Cube_map.svg.png)
- Cubemap 간단한 사용 예시
```
Properties
{
    // 프로퍼티를 만들 때는 Cube를 사용한다.
    _Cube ("CubeMap", Cube) = ""{}
}
SubShader
{
    // 변수 타입은 samplerCUBE를 사용하면 된다.
    samplerCUBE _Cube;

    // 1. 노멀맵이 없는 경우
    struct Input
    {
        float3 worldRefl;   // 월드 반사 벡터
        float3 worldNormal; // 월드 노멀 벡터
    };

    void surf(Input IN, inout SurfaceOutput o)
    {
        // 반사는 단지 주변의 밝음과 어두움을 보여주는 것이므로 Emission에 넣는 것이 옳다.
        // Emission은 Albedo와 합쳐지므로 비율을 주는 것이 좋다.
        // texCUBE(samplerCUBE cube, float3 reflectionVector)
        o.Emission = texCUBE(_Cube, IN.worldRefl).rgb;
    }

    // 2. 노멀맵이 있는 경우
    struct Input
    {
        float3 worldRefl;
        float3 worldNormal;
        INTERNAL_DATA // 픽셀 노멀맵 기반으로 월드 반사 벡터와 노멀 벡터를 생성하게 한다.
    }

    void surf(Input IN, inout SurfaceOutput o)
    {
        o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));

        // 월드 반사 벡터는 WorldReflectionVector()를 사용한다.
        o.Emission = texCUBE(_Cube, WorldReflectionVector(IN, o.Normal)).rgb;
    }
}
```

- 탄젠트 공간
    - 3차원 유클리드 공간에서 표면의 한 점에 접하는 평면을 탄젠트 평면(Tangent Plane)이라고 하며, 이 평면에 대응되는 좌표계를 탄젠트 좌표계라고 한다.

# 16. 알파
- 각 오브젝트에 대한 렌더링 순서는 정해져 있지 않다. 기본적으로 계산이 끝난 오브젝트를 그리게 된다.
- Z Buffering
    - 3D 그래픽스 이미지 심도 좌표 관리 방식
    - 2차원 버퍼에 각 픽셀의 깊이값이 0.0 ~ 1.0으로 저장되어 있다. 0에 가까울 수록 Near Clipping Plane과 가까우며, 1에 가까울 수록 Far Clipping Plane과 가깝다. View Frustum에 있는 객체에 대해서만 계산한다.
    - Z 버퍼를 이용해 현재 픽셀과 새로 그려질 픽셀 중 어떤 것이 관찰자에게 더 가까운지 깊이를 비교할 수 있으며, 이를 이용해 컬링하는 것을 **Z 컬링**이라고 한다.
    - Z 버퍼의 정밀도는 화면 품질에 큰 영향을 미치며 두 물체가 매우 가까울 때 **Z 파이팅** 현상이 일어날 수 있다.
- 반투명 : 알파 블렌딩
    - 반투명의 경우 Z 컬링을 적용하게 되면 의도하지 않은 결과가 나올 수 있다.
    - 해결 방안
        - 불투명(Opaque) 오브젝트는 먼저 그린다.
        > 불투명은 Z 컬링만 해도 올바르게 그려진다.
        - 반투명(Transparent) 오브젝트는 나중에 그린다.
        > 앞에 있는 반투명 오브젝트로 인해 Z 컬링이 발생하는 걸 방지할 수 있다.
        - 반투명 오브젝트끼리는 멀리 있는 것부터 차례대로 그린다.
        > 이를 알파 소팅(Alpha Sorting)이라고 한다.
        - 알파 블렌딩을 사용했을 때는 zwrite를 하지 않는다.
        > 알파 소팅은 카메라와 피봇까지의 거리로 계산이 되는데, 이를 통해서 완벽하게 앞뒤를 구분할 수 없다. 따라서 Z 버퍼에 값을 입력하지 않는다. 하지만 OverDraw가 발생한다.
    - 문제
        - 불투명 오브젝트가 모두 그려질 때까지 대기해야 한다.
        - 알파 소팅을 해야 한다.
        - 앞에 커다란 반투명 오브젝트가 있다면 연산이 많아지게 된다.
        - Deferred Rendering에서는 반투명을 처리하기 어려워서 Forward Rendering으로 반투명을 따로 그려야 한다.



- API
    - [texCUBE()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcube)

- Tip
    - 셰이더 데이터 타입과 정밀도
        - 기본 데이터 타입 : 각 타입마다 정밀도가 달라 성능과 전력 사용량이 다름
            - float
                - 최고 정밀도로 32비트
                - 월드 공간 위치, 텍스쳐 좌표, 복잡한 스칼라 계산 등에 사용
            - half
                - 중간 정밀도로 지원되는 플랫폼에선 16비트, 지원되지 않는 플랫폼에선 float로 변환됨
                - 짧은 벡터, 방향, 오브젝트 공간 위치, HDR 색깔 등에 사용
            - fixed
                - OpenGL ES 2.0에서만 지원하며 11비트이다. 지원되지 않는 플랫폼에선 half나 float로 변환
        - Unity에서는 접미사가 모두 무시되고 `float`로 처리하기 때문에 정밀도 낮은 수를 사용하고 싶다면 변수를 만들어서 처리하는 것이 좋음.
        - 정수는 보통 Loop Counter나 배열 접근을 위해서 사용하는 것이 가장 좋다.
            - 특정 플랫폼(D3D9, OpenGL ES 2.0)에선 지원하지 않아 성능 이슈가 생길 수 있다.
        - 합성 벡터/행렬 타입
            - 벡터
                - 타입 뒤에 숫자를 붙이면 됨
                - 각 요소는 x, y, z, w 혹은 r, g, b, a 로 접근 가능
            - 행렬
                - `float4x4`와 같은 방식으로 정의 가능
                - 일부 플랫폼(OpenGL ES 2.0)에서는 정방행렬만 생성 가능
        - 텍스처/샘플러 타입
            - `sampler2D`와 `samplerCUBE`가 있음
            - 뒤에 `_half`, `_float`를 붙여 절반 정밀도 혹은 최대 정밀도 사용 가능

    - 그래픽은 정확한 것보다 보기 좋은 것이 중요함.
    - [Frame Debugger](https://docs.unity3d.com/2021.3/Documentation/Manual/FrameDebugger.html)을 이용하면 프레임을 그리기 위한 드로우 콜을 볼 수 있다.
    - [SubShader에 Tags 할당하기](https://docs.unity3d.com/2021.3/Documentation/Manual/SL-SubShaderTags.html)
        - RenderPipeline : URP 혹은 HDRP 전용을 표시한다.
        - Queue : Render Queue를 설정한다.
        - RenderType : SubShader의 동작을 변경한다.
        - DisableBatching : Dynamic Batching을 막는다.
        - ForceNoShadowCasting : 그림자 연산을 하지 않는다.
        - PreviewType : Material Inspector 창에서 어떻게 보여질지 결정한다.
        - IgnoreProjector : Projector 컴포넌트의 영향을 받을지, 받지 않을지 결정한다.
    - Transparent Shader는 그림자가 나오지 않는 것이 맞다.
    - Fallback은 해당 Shader를 쓰지 못할 때 대체 Shader를 의미하나, 그림자 패스와도 연관이 있다.
- Reference
    - [Cube Mapping](https://en.wikipedia.org/wiki/Cube_mapping)
    - [Shader Data Types and Precision](https://docs.unity3d.com/Manual/SL-DataTypesAndPrecision.html)
    - [Tangent Space, Tangent Bundle](https://elementary-physics.tistory.com/49)

