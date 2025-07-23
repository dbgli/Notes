#Ubuntu #ROS #Gazebo #RViz #OGRE #Wayland #X11

安装版本：
- Ubuntu 24.04 Noble Numbat
- ROS 2 Jazzy Jalisco
- Gazebo Harmonic

按照官方教程[https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debians.html](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debians.html)安装ROS后，Gazebo和RViz不能创建窗口。具体表现为，运行`gz sim`时报错：
```
[GUI] [Err] [Ogre2RenderEngine.cc:1285]  Unable to create the rendering window: OGRE EXCEPTION(3:RenderingAPIException): currentGLContext was specified with no current GL context in GLXWindow::create at ./.obj-x86_64-linux-gnu/gz_ogre_next_vendor-prefix/src/gz_ogre_next_vendor/RenderSystems/GL3Plus/src/windowing/GLX/OgreGLXWindow.cpp (line 165)
[GUI] [Err] [Ogre2RenderEngine.cc:1293] Unable to create the rendering window after [11] attempts.
[GUI] [Err] [Ogre2RenderEngine.cc:1191] Failed to create dummy render window.
```

以及运行`ros2 run rviz2 rviz2`时报错：
```
[ERROR] [1719902910.679003442] [rviz2]: rviz::RenderSystem: error creating render window: RenderingAPIException: Invalid parentWindowHandle (wrong server or screen) in GLXWindow::create at ./.obj-x86_64-linux-gnu/ogre_vendor-prefix/src/ogre_vendor/RenderSystems/GLSupport/src/GLX/OgreGLXWindow.cpp (line 246)
[ERROR] [1719902910.679011890] [rviz2]: Unable to create the rendering window after 100 tries
terminate called after throwing an instance of 'std::runtime_error'
  what():  Unable to create the rendering window after 100 tries
[ros2run]: Aborted
```

搜索后很快排除了显卡驱动和OpenGL版本的问题，因为我不是虚拟机，没有装在Docker里，也没有英伟达的显卡。有一个可行的解决方案说是OGRE不支持Wayland，加个环境变量强制使用X11就行了，具体如下：

先用`echo $XDG_SESSION_TYPE`确认当前系统使用的是哪种窗口系统，不出意外应该输出`wayland`，然后在`~/.bashrc`里找`QT_QPA_PLATFORM`，有就修改，没有就直接添加`export QT_QPA_PLATFORM=xcb`，最后`source ~/.bashrc`，结束。