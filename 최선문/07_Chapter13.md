# 13. Blinn-Phong 스펙큘러
- PBR 이전에 많이 쓰이던 라이팅 공식은 Lambert를 이용한 Diffuse에 Bling-Phong 스페큘러를 더하고 여기에 Ambient Color를 더하는 것이다.
- [Phong 반사](https://dl.acm.org/doi/pdf/10.1145/360825.360839)는 가장 유명하고 전통적인 스페큘러다.
    - 기본 원리는 **내가 보는 방향으로부터 반사된 방향에 조명이 있으면 그 부분의 하이라이트가 가장 높다**이다.  
        - View 벡터의 반사 벡터(R)와 조명 벡터(L)의 내적
        - 반사 벡터를 구하려면 2N(L*N) - L이라는 공식을 사용하면 된다.
    - 이를 이용해 만들어지는 스페큘러는 원 모양이다. 태양을 상정해서 간략화된 표현이기 때문이다.
- 스페큘러는 언제나 원 모양은 아니다.
- Blinn-Phong : Phong 반사를 좀 더 편리하게 바꾼 것
    - **시선 벡터와 조명 벡터의 중간 값인 하프 벡터를 구하고, 이를 노멀 벡터와 내적한다.**

- 커스텀 라이팅 모델을 만들 때 아래의 인터페이스로도 정의할 수 있다.
```
half4 Lighting<name>(SurfaceOutput o, half3 lightDir, half3 viewDir, half atten);
```

- Blinn-Phong Specular
```
half4 LightingCustomLambert(SurfaceOutput o, half3 lightDir, half3 viewDir, half atten)
{
    half light = dot(o.Normal, lightDir) * 0.5 + 0.5;
    half3 diffColor = o.Albedo * light * _LightColor0.rgb * atten;

    half3 halfVector = normalize(lightDir + viewDir);
    half spec = saturate(dot(halfVector, o.Normal));
    spec = pow(spec, _SpecPower);
    half3 specColor = spec * _SpecCol.rgb;

    half4 final = 0;
    final.rgb = diffColor.rgb + specColor.rgb;
    final.a = o.Alpha;
    
    return final;
}
```

- 결과를 위한 구조체를 임의로 만들 수 있다.
- Rim 라이트를 스페큘러로 활용할 수 있다.


- API

- Tip