# 14. NPR 렌더링
- Blinn Phong의 한계
    - 에너지 보존 법칙을 수동으로 관리해야 한다.
    - 재질에 따른 스펙큘러 색상도 알아서 처리해야 한다.
    - 스펙큘러가 공 모양으로 밖에 표현되지 않아 실제와 많은 차이가 있다.
- 일반적인 Cell Shading의 특징
    1. 외곽선이 있다.
    2. 음영이 끊어져 있다.
- 외곽선을 만드는 방법
    1. 2Pass를 이용한 외곽선 제작
        - 방법
            - 한 번은 외곽선을 그리고, 다른 한 번은 오브젝트를 그린다.
            ```
            SubShader
            {
                // 1Pass : 외곽선을 그린다.

                cull front // 앞면을 컬링한다.

                CGPROGRAM
                // 외곽선을 그리는데 조명과 그림자는 필요 없다.
                #pragma surface surf Nolight noshadow noambient vertex:vert 

                void vert(inout appdata_full v)
                {
                    // 노멀 방향으로 버텍스를 키운다.
                    v.vertex.xyz = v.vertex.xyz + v.normal.xyz;
                }

                // surf 생략

                half4 LightingNolight(SurfaceOutput o, half3 lightDir, half atten)
                {
                    // 검은색으로 그린다.
                    return half4(0, 0, 0, 1);
                }
                ENDCG

                // 2Pass : 오브젝트를 그린다.
                cull back

                // 후략
            }
            ```
        - 장점
            - 균일한 외곽선을 얻을 수 있다.
            - 구현하기 간편하다.
        - 단점
            - 2번 그리는 것이기 때문에 퍼포먼스가 하락한다.
            - 완전히 닫힌 오브젝트가 아닌 경우 뒤집어서 보이게 될 면이 없기 때문에 외곽선이 생기지 않는다.
            - 메시끼리 침범하거나 찌꺼기가 보일 수 있다.
            - 하드 엣지에서는 선이 끊어지게 된다.
    2. Fresnel을 이용한 외곽선 제작
        - 방법
            - Rim 라이트를 만들었던 결과물을 단순히 뒤집어 주거나 조건문으로 외각 라인을 검출한다.
            ```
            // 커스텀 라이트를 사용한다.
            half4 LightingToon(SurfaceOutput o, half3 lightDir, half3 viewDir, half atten)
            {
                // Fresnel로 Rim 라이트를 구한다.
                float rim = abs(dot(o.Normal, viewDir));
                // 조건문으로 음영을 나눈다.
                if (rim > 0.3)
                {
                    rim = 1;
                }
                else
                {
                    // Ambient Color로 밝아지기 때문에 0이 아닌 -1을 사용한다.
                    rim = -1;
                }
            }
            ```
        - 장점
            - 폴리곤의 밀도와 각도에 따라 선의 굵기가 다양하게 나올 수 있다.
        - 단점
            - 완전한 평면을 가진 오브젝트에서는 이상한 모양으로 보이게 된다.
    3. 후처리를 이용한 외곽선 제작

- Diffuse Warping(Ramp Texture)
    - 핵심 구현은 노멀과 라이트 벡터의 내적을 UV로 사용하는 것이다.
    - Ramp Texture는 Clamp 모드로 하고 색상의 손실을 줄이기 위해 압축을 하지 않는다. 대신에 텍스처의 크기를 줄인다.
    ```
    half4 LightingWarpedDiffuse(SurfaceOutput o, half3 lightDir, half3 viewDir, half atten)
    {
        half ndotl = dot(o.Normal, lightDir) * 0.5 + 0.5;
        half4 ramp = tex2D(_RampTex, float2(ndotl, 0.5));

        half4 final;
        final.rgb = o.Albedo.rgb * ramp.rgb;
        final.a = o.Alpha;

        return final;
    }
    ```
    - SSS(Subsurface-Scattering) 효과를 줄 수 있다.
    - 스페큘러에도 활용할 수 있다.
        - tex2D()의 y값이 N dot H를 넣는다.


# Tip
- 그림자 연산은 버텍스 셰이더에서 다룰 수 있는 영역이 아니다.

- Surface Shader Custom Vertex
```
#pragma surface suf Lambert vertex:vert // Optional Parameter에서 버텍스 함수 지정

// signature는 아래와 같음.
void vert(inout appdata_full v)
{
    // 비워놓아도 좌표게 변환은 유니티 엔진이 알아서 해줌
}
```

- 끊어지는 음영 만들기
    - if문으로 하드코딩한다. 음영의 범위를 세부적으로 정할 수 있지만, 부하가 크다.
    ```
    float someValue;
    if (someValue > 0.5)
    {
        someValue = 1;
    }
    else if (someValue > 0.3)
    {
        someValue = 0.4;
    }
    else
    {
        someValue = 0;
    }
    ```
    - ceil()을 이용한다. 간단하지만, 음영의 세부적인 범위를 정할 순 없다.
    ```
    float someValue;
    someValue = someValue * 5;          // 범위를 5배 키우고
    someValue = ceil(someValue) / 5;    // ceil()로 계단식 값을 만든 다음 0 ~ 1의 범위가 되도록 5를 다시 나눈다.
    ```


# API & Manual
- [ShaderLab: commands](https://docs.unity3d.com/Manual/shader-shaderlab-commands.html)
- [Surface Shader Custom Modifier Functions](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)
- [Vertex Data](https://docs.unity3d.com/Manual/SL-VertexProgramInputs.html)
- [HLSL: if](https://learn.microsoft.com/ko-kr/windows/win32/direct3dhlsl/dx-graphics-hlsl-if)
- [Illustrative Rendering in Team Fortress 2](https://cdn.cloudflare.steamstatic.com/apps/valve/2007/NPAR07_IllustrativeRenderingInTeamFortress2.pdf)