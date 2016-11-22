---
layout: post
title:  "用shader实现Photoshop中图层叠加混合模式效果"
date:   2016-11-22 20:35
categories: jekyll
permalink: /archivers/additive-effect-in-shader
published: true
---
&emsp;&emsp;今天美术同学问我游戏引擎里（cocos2d-x）能不能实现Photoshop里一个叫做叠加的图层混合模式效果，我本来想应该只是简单设置一下Sprite的BlendFunc就可以了，但其实没那么简单。


&emsp;&emsp;首先Photoshop里说的叠加和BlendFunc里的additive模式是不一样的。引擎里的additive模式如下代码，只是简单的根据源颜色的alpha分量来混合目标颜色得到最终的颜色。

~~~cpp
const BlendFunc BlendFunc::ADDITIVE = {GL_SRC_ALPHA, GL_ONE};
~~~

Photoshop里的图层叠加混合稍微复杂一点，计算公式：

&emsp;&emsp;基色 < = 128：结果色 = 混合色 * 基色 / 128；
&emsp;&emsp;基色 > 128：结果色 = 255 - （255 - 混合色）* (255 - 基色) / 128

相当于是根据混合目标颜色的每个分量值来选择不同公式计算得到最终的颜色值分量。
为此写了一个特定的fragment shader来实现Photosh里的这个叠加效果，其中background_texture为混合的目标纹理。

~~~cpp
varying vec4 v_fragmentColor;
varying vec2 v_texCoord;
uniform sampler2D background_texture;
void main()
{
    vec4 texColor0 = texture2D(CC_Texture0, v_texCoord);
    texColor0 = v_fragmentColor*texColor0;
    vec4 texColor1 = texture2D(background_texture, v_texCoord);
    vec4 color = vec4(texColor0.r,texColor0.g,texColor0.b,texColor0.a);
        
    if(texColor1.r<=0.5){
        color.r=(texColor1.r*texColor0.r)/0.5;
    } else {
        color.r=1.0-(1.0-texColor1.r)*(1.0-texColor0.r)/0.5;
    }
    if(texColor1.g<=0.5){
        color.g=(texColor1.g*texColor0.g)/0.5;
    } else {
        color.g=1.0-(1.0-texColor1.g)*(1.0-texColor0.g)/0.5;
    }
    if(texColor1.b<=0.5){
        color.b=(texColor1.b*texColor0.b)/0.5;
    } else {
        color.b=1.0-(1.0-texColor1.b)*(1.0-texColor0.b)/0.5;
    }
    if(texColor1.a<=0.5){
        color.a=(texColor1.a*texColor0.a)/0.5;
    } else {
        color.a=1.0-(1.0-texColor1.a)*(1.0-texColor0.a)/0.5;
    }
    gl_FragColor = color;
}
~~~