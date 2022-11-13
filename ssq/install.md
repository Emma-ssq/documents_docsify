# webui安装与更新

## setup.py

**调用nni源码中的setup.py文件**, `python setup.py 参数`

1. Develop类中执行`setup_ts.build(release=None)`

2. **setup_ts.build()**

   ```python
   def build(release):
       """
       Compile TypeScript modules and copy or symlink to nni_node directory.
   
       `release` is the version number without leading letter "v".
   
       If `release` is None or empty, this is a development build and uses symlinks on Linux/macOS;
       otherwise this is a release build and copies files instead.
       On Windows it always copies files because creating symlink requires extra privilege.
       """
       if release or not os.environ.get('GLOBAL_TOOLCHAIN'):
           download_toolchain()
       # 将toolchain/node中的文件复制到nni_node文件夹中
       prepare_nni_node()
       # 为jupyterlab兼容更新相关包版本
       update_package()
       # compile TypeScript code
       compile_ts(release)
       # a release build
       if release or sys.platform == 'win32':
           copy_nni_node(release)
       else:
           symlink_nni_node()
       restore_package()
   ```

## 前端修改逻辑

主要修改`ts/webui/`文件夹中的东西。

修改完成后更新操作：

1. 执行`pip setup.py develop`,重新编译typescripts文件，
2. 更新上传包：
   - `export NNI_RELEASE=2.0`，当然2.0这个数字是可以不断修改的。
   - `python setup.py build_ts`
   - `python setup.py bdist_wheel` 打包在dist文件中，可以传播下载。
3. 删除包`pip install nni` 或者`python setup.py clean [--all]` 其中all是用来删除wheel文件的。
