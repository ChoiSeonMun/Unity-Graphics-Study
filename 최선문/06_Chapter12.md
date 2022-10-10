# 12. Rim 라이트
- 모든 물체는 기울일 수록 반사가 심해진다.
- 반사율은 재질에 따라 다르다.
    - 물리 기반 쉐이더는 이러한 반사율을 BRDF(Bidirectional Reflectance Distribute Function)을 이용해 구현됨.
    - 반사 공식은 Fresnel 이라고 한다.
- 만약 역광이 있을 때, 털이나 반투명의 재질을 가진 물체는 이 현상이 매우 강렬하게 나타날 수 있고, 재질에 따라 때로는 잘 안보이게 될 수 있다. 이를 Rim Light라고 한다.
- 게임에서는 배경과 캐릭터의 분리나 강조를 위해서 혹은 선택되었을 때나 특정 상태를 표현하기 위한 이펙트 또는 셀 셰이딩 효과 같은 특수한 효과를 위해 과장되도록 사용한다.
- [참고자료](https://www.studiobinder.com/blog/what-is-a-rim-light-photography-definition/)

- Rim Light (Simple Fresnel)
```
float _RimPower;
float rim = pow(1 - dot(o.Normal, IN.viewDir), _RimPower);

float4 _RimColor;
o.Emission = rim * _RimColor;
```

- Rim Light를 알파 채널에 넣으면 홀로그램 효과를 만들 수 있다.

- Hologram (using rim light)
```
void surf (Input IN, inout SurfaceOutput o)
{
    o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));
    
    float rim = saturate(dot(o.Normal, IN.viewDir));
    rim = pow(1 - rim, _RimPower);
    
    float stripe = pow(frac(IN.worldPos.g * _StripeFreq - _Time.y * _StripeSpeed), _StripeWidth);
    
    o.Alpha = rim + stripe;
    o.Emission = _Color;
}
```


- API
    - [abs()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-abs) : 절댓값
    - [frac()](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-frac) : 소수점
- 자잘한 팁
    - 1. rgb를 검게 만들어 놓고 테스트를 하면 효과적이다.
    - 2. Fallback은 본래 그래픽 카드에서 셰이더 연산을 실패할 때 대체 셰이더의 이름을 적는 란이나, 그림자를 연산할 때 Fallback에 씌여진 셰이더가 영향을 미친다. => Fix 되었는지는 확인 필요

