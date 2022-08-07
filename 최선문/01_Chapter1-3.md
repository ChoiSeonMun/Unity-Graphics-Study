# 1. 쉐이더(Shader)란 무엇인가?
**쉐이더란 무엇인가?**
* Shader : 3D 컴퓨터 그래픽에서 최종적으로 화면에 출력하는 픽셀의 색을 정해주는 함수

# 2. 쉐이더를 익히기 위한 기반 지식들
**렌더링 파이프라인**
1. 오브젝트 데이터 받아오기
    * 버텍스(Vertex)로 이루어진 물체의 데이터 값을 받아온다.
        * Vertex : 물체를 구성하는 점으로 렌더링을 하기 위한 속성(attribute)을 갖고 있다.
            * Ex) Index Number, Position, Normal, Color, UV etc.
    * 버텍스를 가지고 삼각형을 만들어 오브젝트의 기본적인 형태를 만든다.

2. 버텍스 쉐이더(Vertex Shader)
    * 각종 버텍스와 관련된 연산을 할 수 있다.
    * 아래와 같은 좌표 변환을 한다.
        * 로컬 좌표계에서 월드 좌표계로 변환
        * 월드 좌표계에서 뷰 좌표계로 변환
        * 뷰 좌표계에서 Perspective Project 변환

3. 레스터라이저(Rasterizer)
    * 3D 공간을 2D인 모니터로 옮기는 과정

4. 픽셀 쉐이더 / 프래그먼트 쉐이더
    * 조명과 텍스처, 그림자와 각종 특수효과 등을 연산

[OpenGL의 렌더링 파이프라인](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview)
[Direct3D 11의 렌더링 파이프라인](https://docs.microsoft.com/ko-kr/windows/win32/direct3d11/overviews-direct3d-11-graphics-pipeline)

**Unity에서 제공하는 렌더 파이프라인**
Unity의 Render Pipeline은 Built-in Render Pipeline과 Scriptable Render Pipeline(SRP)으로 나뉜다.

* Built-in Render Pipeline
    * 범용으로 사용되나 커스터마이즈 옵션이 제한적임

* Universal Render Pipeline
    * 광범위한 플랫폼에서 최적화된 그래픽스를 구현하도록 지원하는 SRP

* High Definition Render Pipeline
    * 고사양 플랫폼을 위한 최신 고해상도 그래픽스를 구현하도록 지원하는 SRP

**모니터에 표현되는 색과 빛의 기본 원리**
* 모니터는 가산혼합을 사용해서 색을 만듦.
* 컬러 연산엔 덧셈, 뺄셈, 곱셈, 나눗셈, 반전이 있다.

# 3. 코딩의 기본 / 유니티 쉐이더 구성
* Unity는 ShaderLab이라는 멀티플랫폼을 대응하기 위해 자체 스크립트 언어를 사용하고 있음q
* ShaderLab을 이용한 제작 방식
    * ShaderLab으로만 제작 => 지원 중단됨
    * Surface Shader로 제작
        > - 기본적인 조명과 버텍스 쉐이더의 복잡한 부분은 스크립트를 이용하여 자동으로 처리되며, 간단히 픽셀 쉐이더 부분만 작성할 수도 있음.
        > - 최적화에는 다소 무리가 있고, 일정 수준 이상의 고급 기법 구현은 어려움
    * Vertex & Fragment Shader로 제작
        > 버텍스 쉐이더부터 일일이 제작
