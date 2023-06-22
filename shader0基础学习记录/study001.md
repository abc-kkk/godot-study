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

