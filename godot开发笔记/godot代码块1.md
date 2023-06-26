await 等待
```gdscript
var time = get_tree().create_timer(3)
await time.timeout
print("a")
```

设置 `Camera2d` 边界代码块
```gdscript
# 地图在tailmap上占用了多大的范围（以图款为单位）
var used := tile_map.get_used_rect().grow(-1)
# 一个图块的大小
var tile_size := tile_map.tile_set.tile_size

camera_2d.limit_top = used.position.y * tile_size.y
camera_2d.limit_left = used.position.x * tile_size.x
camera_2d.limit_bottom = used.end.y * tile_size.y
camera_2d.limit_right = used.end.x * tile_size.x
camera_2d.reset_smoothing()
```

打开关闭 碰撞检测功能
```gdscript
coll_shape.disable = true/false
```

切换场景
```gdscript
get_tree().change_scene("res://xxx.tscn")
```

关闭游戏
```gdscript
get_tree().quit()
```

两个方向用Input.get_axis
四个方向用Input.get_vector

shader每个独占一个shader文件的用法
```
_shader_material = ShaderMaterial.new()
_shader_material.shader = mat
enemy.material = _shader_material
_shader_material.set_shader_parameter("flash_modifier",0)
```

