可以用tween 控制shader中的变量值的改变。
比如iTime在3秒内从0-1
"material:shader_param/iTime"
实现技能特效发光
```
shader_type canvas_item;
uniform sampler2D emission_texture;
uniform vec4 offset_color: source_color = vec4(1.0,0.0,0.0,1.0);

void fragment() {
	
	vec4 current_color = texture(TEXTURE,UV);
	vec4 emission_color = texture(emission_texture,UV);
	if (emission_color.r > 0.0) {
//		emission_color = emission_color + vec4(1.0,1.0,1.0,1.0);
//COLOR = current_color;
		COLOR = emission_color + offset_color;
	}else{
		COLOR = current_color;
	}
}
```
需要ps改变背景为黑色 ，教程 https://www.bilibili.com/video/BV1R54y197yW/?vd_source=be05daba11f7085e625137b96439f0f3 
粒子有两个着色器，一个是粒子着色器， 可以使用系统的图形界面 .shader_type是 particles  , 只有顶点着色器
一个是canvas_item 



