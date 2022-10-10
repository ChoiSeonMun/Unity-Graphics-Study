# 8. SurfaceOutputStandard 사용하기
- [관련 매뉴얼](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)
- Unity 5부터 PBR 지원

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

struct SurfaceOutputStandardSpecular
{
    fixed3 Albedo;      // diffuse color
    fixed3 Specular;    // specular color
    fixed3 Normal;      // tangent space normal, if written
    half3 Emission;
    half Smoothness;    // 0=rough, 1=smooth
    half Occlusion;     // occlusion (default 1)
    fixed Alpha;        // alpha for transparencies
};
```

- Smoothness는 매끄러운지 거친지를 결정하며, 0이면 난반사만 발새하고, 1이면 정반사만 일어남.
- [MaterialCharts](https://docs.unity3d.com/Manual/StandardShaderMaterialCharts.html)
- 표준 셰이더에서는 텍스쳐를 가지고 Metalic과 Smoothness를 조정할 수도 있음
- Normal Mapping
- [노멀맵에 대하여](https://docs.unity3d.com/kr/current/Manual/StandardShaderMaterialParameterNormalMap.html)
- DXT1 / DXT5가 텍스처 표준
- NormalMap은 DXTnm 포맷임.
- NormalMap은 UnpackNormal() 이용
- [Surface Shader Example](https://docs.unity3d.com/Manual/SL-SurfaceShaderExamples.html)

- [Ambient Occlusion](https://www.sciencedirect.com/topics/computer-science/ambient-occlusion)(환경 차폐)

# 9. 버텍스 컬러를 이용해봅시다
- 버텍스에는 위치(Position), UV(Texcoord), 컬러(Color), 노멀(Normal), 탄젠트(Tangent) 등이 있다.
- 컬러의 기본값은 float4(1, 1, 1, 1)이다.
- 버텍스 컬러를 설정하려면 그래픽 툴에서 그리거나, 엔진의 자체 기능을 이용한다.
