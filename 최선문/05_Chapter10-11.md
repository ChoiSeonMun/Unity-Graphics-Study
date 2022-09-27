# 10. 디지털 라이트 이론
- 디지털 라이트의 분류
    - Directional Light
        - 직진성을 가진 조명
        - 조명의 강도와 컬러 등의 정보를 제외하면 방향 밖에 없음
        - 시작점과 끝점도 없고, 조명을 조사하는 넓이의 개념이 없어 조명 연산 중 가장 가벼운 조명임
        - 주로 태양을 시뮬레이트 하거나 그림자를 만들 때 주로 사용된다.
    - Point Light
        - 점 모양의 광원으로 사방으로 뻗어 나가는 특성이 있다.
        - 둥근 모양의 조명 혹은 일정 범위의 국지적 분위기를 표현하기 위한 목적으로 사용 됨.
            - Ex. 등불, 스파크나 폭발
        - Directional Light와 비교하여 무겁기 때문에 성능에 신경쓰며 작업해야 함.
    - Spot Light
        - 원뿔 모양의 라이트로 특정한 부분을 강조하거나 표현할 때 사용함.
            - Ex. 손전등, 차 헤드라이트, 등대 등
        - 성능을 생각하면서 사용해야 함.
    - Unity의 기본 라이트 타입은 [여기](https://docs.unity3d.com/Manual/Lighting.html)서 확인할 수 있다.
- 디지털 라이트는 물체의 법선 벡터와 광원의 방향 벡터의 각도를 이용해 연산이 이뤄진다.
    - 마주볼 수록 가장 강한 빛을 받아 밝아지며, 수직이면 가장 적은 빛을 받아 어두워진다.
- 법선 벡터는 버텍스가 갖고 있으며, 이를 이용해 조명 연산을 하고, 버텍스 사이의 밝기 값은 보간한다.
    - 이렇게 하는 이유는 법선 벡터가 면에 있으면 부드러운 라이팅을 구현할 수 없기 때문이다.
    - 이를 버텍스 라이팅(Vertex Lighting)이라고 한다.
    - 이 때문에 각진 형태를 표현하려면 같은 위치에 다른 법선 벡터를 가진 버텍스를 만들어 줘야 한다.
        - 버텍스는 하나의 법선 벡터를 가지기 때문이다.
- 자잘한 팁
    - 1. 길이가 같은 두 벡터를 더하면 두 벡터 사이의 절반인 각도가 나온다.
    - 2. 조명 연산을 할 때는 조명 벡터를 뒤집어서 연산하면 편하다.
        - Unity에서는 뒤집어서 들어온다.

# 11. Lambert 라이트
- 표면 셰이더에 커스텀 라이팅 모델을 사용하고 싶으면 [여기](https://docs.unity3d.com/Manual/SL-SurfaceShaderLighting.html) 문서를 참고하면 된다.

```


half4 Lighting<Name>(SurfaceOutput o, half3 viewDir, half atten) { }

// SurfaceOutput
struct SurfaceOutput
{
    fixed3 Albedo;  // diffuse color
    fixed3 Normal;  // tangent space normal, if written
    fixed3 Emission;
    half Specular;  // specular power in 0..1 range
    fixed Gloss;    // specular intensity
    fixed Alpha;    // alpha for transparencies
};

// viewDir : 조명 방향 벡터로 반대로 뒤집어서 들어온다.

// atten : 감쇠
```

- Half-Lambert
    - Lambert는 가볍지만 cos 그래프의 특성상 밝다가 너무 갑자기 검게 음영이 떨어지게 된다.
    - 이를 해결하고자 한 것이 [Half-Lambert](https://developer.valvesoftware.com/wiki/Half_Lambert)이다.
    - [밸브에서 발표했다.](https://cdn.akamai.steamstatic.com/apps/valve/2006/SIGGRAPH06_Course_ShadingInValvesSourceEngine.pdf)
    - Half-Lambert 공식
        - Lambert 값에 * 0.5 + 0.5를 한다.
            - 이 경우 빛이 180도인 부분까지 영향을 미치게 되어 부자연스러울 수 있다. 그래서 여기에 3제곱을 하면 좀 더 현실에 가까운 음영이 나오게 된다.
        - Lambert 값에 * 2 - 1을 한다.

- Lambert 공식 : Unity의 Legacy Shaders/Bumped Diffuse와 완전히 같은 쉐이더
```
half4 LightingTest (SurfaceOutput o, half3 lightDir, half atten) {
    half ndotl = saturate(dot(o.Normal, lightDir)); // 빛의 방향에 따른 밝기를 구한다.
    half4 result;
    result.rgb = ndotl * o.Albedo * _LightColor0.rgb * atten; // 텍스처와 조명의 컬러, 감쇠를 적용한다.
    result.a = o.Alpha; // 알파값을 적용한다.
    return result;
}
```

- 감쇠를 고려하지 않으면 아래와 같은 일이 일어나지 않는다.
    - self-shadow가 생기지 않는다.
    - receive shadow가 동작하지 않는다.
    - 조명의 감쇠 현상이 일어나지 않는다.

- 자잘한 팁
    - 1. 조명 연산을 할 때 음수는 의도치 않은 결과를 불러일으킬 수 있다.
        - Ex. 색이 밝아지지 않는다던가
    - 2. [Unity 내장 변수](https://docs.unity3d.com/Manual/SL-UnityShaderVariables.html)는 여기서 확인할 수 있다.

        
- API
    - [dot()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-dot) : 내적
    - [saturate()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-saturate) : 0 ~ 1 사이의 값으로 만들어 줌
    - [max()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-max) : 큰 값
    - [pow()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-pow) : 제곱


