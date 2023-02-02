## 1. 编写URDF

1. 编写相机的link
2. 添加插件

```markup
<robot>
  ... robot description ...
  <link name="camera_link">
    ... link description ...
  </link>

  <gazebo reference="camera_link">
    <sensor type="camera" name="camera">
      ... sensor parameters ...
      <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
        ... plugin parameters ..
      </plugin>
    </sensor>
  </gazebo>

</robot>
```