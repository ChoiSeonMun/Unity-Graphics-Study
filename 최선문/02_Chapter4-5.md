# 4. 기초적인 서피스 쉐이더를 제작해 봅시다.
- 서피스 쉐이더는 세 부분으로 나눌 수 있다.
- 쉐이더의 이름을 작성하는 부분
```
// 아래의 이름은 '/'에 따라 Unity 에디터에서
// 트리 구조를 만들어 준다.
Shader "Custom/NewSurfaceShader"
{

}
```
- 쉐이더의 인터페이스를 만드는 부분.
- https://docs.unity3d.com/kr/current/Manual/SL-Properties.html
```
Properties
{
    <Material property declaration>
    <Material property declaration>
}

[optional: attribute] name("display text in Inspector", type name) = default value
```

- 서브쉐이더에는 실제 쉐이더가 작성된다. HLSL 코드가 들어간다.
```
SubShader
{
    <optional: LOD>
    <optional: tags>
    <optional: commands>
    <One or more Pass definitions>
}
```

- 서피스 쉐이더를 사용하는 경우 아래 전처리기 사용
```
#pragma surface <surface function> <lighting model> <optional parameters>
```

- 변수를 만드는 것은 C와 유사함.
- 타입 : https://docs.microsoft.com/ko-kr/windows/win32/direct3dhlsl/dx-graphics-hlsl-data-types

- SurfaceOutputStandard
```
struct SurfaceOutputStandard
{
    fixed3 Albedo;      // base (diffuse or specular) color
    fixed3 Normal;      // tangent space normal, if written
    half3 Emission;     
    half Metallic;      // 0=non-metal, 1=metal
    half Smoothness;    // 0=rough, 1=smooth
    half Occlusion;     // occlusion (default 1)
    fixed Alpha;        // alpha for transparencies
};
```

- 색상의 출력은 클램핑 되나, 실제 값은 0~1 범위를 넘어설 수 있다.

# 5. Surface Shader를 이용해서 텍스처를 제어해봅시다.
- 내장 함수 : https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions

- tex2D() : https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2d

- Grayscale 방식 : https://www.baeldung.com/cs/convert-rgb-to-grayscale