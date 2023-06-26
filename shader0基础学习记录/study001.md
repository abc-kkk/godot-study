在 godot 中使用shader笔记

一个闪白效果
```shader
shader_type canvas_item;

// 通过uniform 让引擎脚本访问我们的变量
uniform vec4 flash_color: source_color = vec4(1.0);
uniform float flash_modifier: hint_range(0.0, 1.0) = 1;

void fragment() {
	vec4 color = texture(TEXTURE,UV);
	// mix 把挡墙颜色混合成想要的颜色 , 前两个参数是我们想要混合的颜色，最后一个参数是我们想让第二个值和第一个值混合的程度
	// 这里素材图像是rgb，背景是透明色，所以只有人物变成了白色
	color.rgb = mix(color.rgb,flash_color.rgb,flash_modifier);
	COLOR = color;
}
```

在GDScript中用法
```gdscript
@onready var sprite_2d: Sprite2D = $Sprite2D
@onready var mt = sprite_2d.material as ShaderMaterial
@onready var timer: Timer = $Timer

func _process(delta: float) -> void:
	if Input.is_action_just_pressed("space"):
		mt.set_shader_parameter("flash_modifier",1)
		timer.start(0.1)
		
func _on_timer_timeout() -> void:
	mt.set_shader_parameter("flash_modifier",0)
```

上述的`UV` 是一个`vec2`类型的 可读可写  
```
inout vec2 UV
inout vec4 COLOR
sample2D TEXTURE  // 这个 TEXTURE 其实就是我们的图片资源 ，没啥稀奇的
```


```
// 顶点着色器 ，可以操纵顶点
void vertex(){

}
// 片段着色器，在平面上表示每个像素 ，每个像素在多线程的调用 这个函数。也可以叫像素着色器
void fragment(){

}
```

`COLOR = texture(TEXTURE,UV)` 显示纹理

通过操作UV对图片平移旋转缩放
```
shader_type canvas_item;

uniform float angle = 0.0;
uniform sampler2D t_image;
// 可以从顶点着色器到片段着色器传递

varying float h ;

// 可以执行4次 因为有四个顶点
void vertex(){
	h = 0.5;
}
void fragment() {
	vec2 _uv = UV ; 
	// _uv += 0.1; 异动UV 
//	_uv.x += 0.5;
	// 加减法平移， 乘除法 缩放
	// 旋转是乘以 2*2 矩阵 mat2
	mat2 m = mat2(vec2(cos(angle),-sin(angle)),vec2(sin(angle),cos(angle)) );
	_uv = _uv * m;
	// 这里可以使用h
	COLOR = texture(t_image,_uv);
//	COLOR = vec4(_uv,0.0,1.0);
```


水平翻转和垂直翻转
```
_uv.x = 1.0 - _uv.x;
_uv.y = 1.0 - _uv.y;
```


好的着色器，都需要很强的算法，所以 网上抄 会用就行，自己能写简单的就行。好游戏都是偷来的
一个实现帧动画的效果
```
shader_type canvas_item;

vec2 flipbook(vec2 uv, vec2 size, float progress){
	progress = floor( mod(progress, (size.x * size.y)) );
	vec2 frame_size = vec2(1.0, 1.0) / vec2(size.x, size.y);
	vec2 frame = fract(uv / size) + frame_size;
	
	frame.x += ( (progress / size.x) - frame_size.x * floor(progress/size.x) * size.x ) - frame_size.x;
	frame.y += (frame_size.y * floor(progress/size.x) ) - frame_size.y ;
	
	return frame;
}

void fragment()
{
	vec2 flip_uv = flipbook(UV, vec2(18, 1), TIME * 10.);
	vec4 flip_texture = texture(TEXTURE, flip_uv);
	COLOR = flip_texture;
}
```



```

void fragment()
{
	vec2 flip_uv = flipbook(UV, vec2(18, 1), TIME * 10.);
	vec4 flip_texture = texture(TEXTURE, flip_uv);
	// 这里如果是黑色背景 可以通过 修改a通道
	// 也可以加offset 调整颜色
	flip_texture = vec4(flip_texture.r*color_offset,flip_texture.g*color_offset,flip_texture.b*color_offset,flip_texture.r);
	
	COLOR = flip_texture;
}
```

圆形和圆角遮罩 ， 可以用来切图?

```
shader_type canvas_item;

uniform sampler2D img;

float circle(vec2 position, float radius, float feather)
{
	return smoothstep(radius, radius + feather, length(position - vec2(0.5)));
}

vec4 square_rounded(vec2 uv, float width, float radius){
	uv = uv * 2.0 - 1.0;
	
	radius *= width; // make radius go from 0-1 instead of 0-width
	vec2 abs_uv = abs(uv.xy) - radius;
	vec2 dist = vec2(max(abs_uv.xy, 0.0));
	float square = step(width - radius, length(dist));
	return vec4(vec3(square), 1.0);
}

void fragment() {
	
	/*
	// 圆形遮罩
	vec2 uv = UV;
	uv -= 0.5;
	float d = length(uv)*3.0;
	clamp(d,0.0,1);
	COLOR = vec4(d,0.0,0.0,1.0); 
	
	vec4 color_tex = texture(img,UV);
	COLOR = vec4(color_tex.rgb,1.0-d);
	*/
	vec4 color_tex = texture(img,UV);
	// 圆形
//	 COLOR = vec4(color_tex.rgb ,1.0 - circle(UV, 0.4, 0.055));
	// 圆角矩形
//	COLOR = square_rounded(UV, 1, 0.5);
	COLOR = vec4(color_tex.rgb ,1.0 - square_rounded(UV, 0.6, 0.8).r);	
}
```

在 https://www.shadertoy.com/view/ldBXz3 上抄写代码 修改适配godot的例子 
```
shader_type canvas_item;

uniform float iTime;
uniform vec3 iResolution;


const float samp = 30.0;

float grid(vec2 p) {
  vec2 orient = normalize(vec2(1.0,3.0));
  vec2 perp = vec2(orient.y, -orient.x);
  float g = mod(floor(1. * dot(p, orient)) + floor(1. * dot(p, perp)), 2.);
  return g;
}


vec4 mainImage(in vec4 fragCoord )
{
	float time = iTime*0.5;
  	vec2 p = fragCoord.xy / 50. + vec2(-time, time);
  	vec2 q = (fragCoord.xy - (iResolution.xy / 2.)) / iResolution.x / 1.5 ;
  	vec4 c = vec4(grid(p));
  	if (q.x + 0.1 * q.y > 100.) {
    	return c;
  	}
  	else {
	    vec4 cc = vec4(0.0);
	    float total = 0.0;
	    
	    float radius = length(q) * 100.;
	    for (float t = -samp; t <= samp; t++) {
	      float percent = t / samp;
	      float weight = 1.0 - abs(percent);
		  float u = t / 100.;
	      vec2 dir = vec2(fract(sin(537.3 * (u + 0.5)) ) , fract(sin(523.7 * (u + 0.25)) ));
	      dir = normalize(dir) * 0.01;
	      float skew = percent * radius;
	      vec4 samplev = vec4(
	          grid(vec2(0.03,0.) + p +  dir * skew),
	          grid(radius * vec2(0.005,0.00) + p +  dir * skew),
	          grid(radius * vec2(0.007,0.00) + p +  dir * skew),
	          1.0);
	      cc += samplev * weight;
	      total += weight;
    	}
    	return cc / total - length(q ) * vec4(1.,1.,1.,1.) * 1.5;
  }
}


void fragment() {
	COLOR = mainImage(FRAGCOORD);
}

```


给图片加上一条白色烟飘过的特效 
https://www.shadertoy.com/view/mdfcW7

```
shader_type canvas_item;

uniform float iTime;
uniform vec3 iResolution;
uniform sampler2D img;

float getColor(float y, float time, float diff, float speed, float seed, float mult)
{
    return y + sin((time + diff) * speed + seed) * mult;
}

vec3 getSplitColor(vec2 uv, float m, float seed)
{
    float speed = 1.2;
    float xMult = 20. + seed * 10.;
    float diff = 0.02;
    float time = iTime + uv.x * xMult * m + cos(uv.x+iTime/5.)*10.;
    time *= 0.5;
    float r = getColor(uv.y, time, -diff, speed, seed, m);
    float g = getColor(uv.y, time, 0., speed, seed, m);;
    float b = getColor(uv.y, time, diff, speed, seed, m);;
    return smoothstep(20./iResolution.y, 0., abs(vec3(r, g, b)));
}

vec4 mainImage(  in vec4 fragCoord )
{
    vec2 uv = (2.0*fragCoord.xy-iResolution.xy)/iResolution.y/1.2;    
    vec3 col = vec3(0);
    float decay = cos(uv.x)/(3.14*2.)*1.5;

    float iter = 50.;
    for(float i=0.;i<iter;i++)
    {
        col += getSplitColor(uv, decay, (i*1.5)/iter);
    }

    return vec4(col/(iter/4.),1.0);
}

void fragment() {
	// Place fragment code here.
	vec4 color_tex = texture(img,UV);
	vec4 effect = mainImage(FRAGCOORD);
	effect = vec4(effect.rgb,effect.r);
	COLOR = color_tex + effect ; // vec4(color_tex.rgb,(1.0 - effect.r));
}
```

也许可以用来做闪电链

```
shader_type canvas_item;

uniform float iTime;
uniform vec3 iResolution;
float rand(vec2 n) {
    return fract(sin(dot(n, vec2(12.9898, 4.1414))) * 43758.5453);
}

float noise(vec2 n) {
    const vec2 d = vec2(0.0, 1.0);
    vec2 b = floor(n), f = smoothstep(vec2(0.0), vec2(1.0), fract(n));
    return mix(mix(rand(b), rand(b + d.yx), f.x), mix(rand(b + d.xy), rand(b + d.yy), f.x), f.y);
}

float fbm(vec2 n) {
    float total = 0.0, amplitude = 1.0;
    for (int i = 0; i < 7; i++) {
        total += noise(n) * amplitude;
        n += n;
        amplitude *= 0.5;
    }
    return total;
}

vec4 mainImage( in vec4 fragCoord )
{
    vec4 col = vec4(0,0,0,1);
    vec2 uv = fragCoord.xy * 1.0 / iResolution.xy;
   
   
    // draw a line, left side is fixed
    vec2 t = uv * vec2(2.0,1.0) - iTime*3.0;
    vec2 t2 = (vec2(1,-1) + uv) * vec2(2.0,1.0) - iTime*3.0; // a second strand
   
    // draw the lines,
//  this make the left side fixed, can be useful
//  float ycenter = mix( 0.5, 0.25 + 0.25*fbm( t ), uv.x*4.0);
//    float ycenter2 = mix( 0.5, 0.25 + 0.25*fbm( t2 ), uv.x*4.0);
    float ycenter = fbm(t)*0.5;
    float ycenter2= fbm(t2)*0.5;

    // falloff
    float diff = abs(uv.y - ycenter);
    float c1 = 1.0 - mix(0.0,1.0,diff*20.0);
   
    float diff2 = abs(uv.y - ycenter2);
    float c2 = 1.0 - mix(0.0,1.0,diff2*20.0);
   
    float c = max(c1,c2);
    col = vec4(c*0.6,0.2*c2,c,1.0); // purple color
    return col;
}
void fragment() {
	// Place fragment code here.
	
	COLOR = mainImage(FRAGCOORD);
	
}
```


vec4 lighten(vec4 base, vec4 blend){
	return max(base, blend);
}
这个函数可以实现叠加闪电效果


```
vec2 random(vec2 uv) {
	return vec2(fract(sin(dot(uv.xy,
		vec2(12.9898,78.233))) * 43758.5453123));
}

float worley(vec2 uv, float columns, float rows) {
	
	vec2 index_uv = floor(vec2(uv.x * columns, uv.y * rows));
	vec2 fract_uv = fract(vec2(uv.x * columns, uv.y * rows));
	
	float minimum_dist = 1.0;  
	
	for (int y= -1; y <= 1; y++) {
		for (int x= -1; x <= 1; x++) {
			vec2 neighbor = vec2(float(x),float(y));
			// vec2 point = random(index_uv + neighbor);
			vec2 point = random(index_uv + neighbor);
float speed = 4.5;
point = vec2( cos(TIME * point.x * speed), sin(TIME * point.y * speed) ) * 0.5 + 0.5;
			
			vec2 diff = neighbor + point - fract_uv;
			float dist = length(diff);
			minimum_dist = min(minimum_dist, dist);
		}
	}
	
	return minimum_dist;
}

vec2 voronoi(vec2 uv, float columns, float rows) {
	
	vec2 index_uv = floor(vec2(uv.x * columns, uv.y * rows));
	vec2 fract_uv = fract(vec2(uv.x * columns, uv.y * rows));
	
	float minimum_dist = 1.0;  
	vec2 minimum_point;

	for (int y= -1; y <= 1; y++) {
		for (int x= -1; x <= 1; x++) {
			vec2 neighbor = vec2(float(x),float(y));
			vec2 point = random(index_uv + neighbor);

			vec2 diff = neighbor + point - fract_uv;
			float dist = length(diff);
			
			if(dist < minimum_dist) {
				minimum_dist = dist;
				minimum_point = point;
			}
		}
	}
	return minimum_point;
}
void fragment() {
	vec3 voronoi = vec3(voronoi(UV, 10.0, 5.0).r);
	COLOR = vec4(voronoi, 1.0);
}
```




