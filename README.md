# hugsim-runtime

`hugsim-runtime` 是 HUGSIM 使用的第三方 Pixi 运行环境仓库。它只管理 Python、PyTorch/CUDA wheel、第三方源码依赖和后置安装任务，不安装 HUGSIM 项目自己的 `sim/` 包。

## 目录结构

```text
hugsim-runtime/
  pixi.toml
  pixi.lock
  external/
  torch_whl/
```

`external/` 下的第三方源码通过 Git submodule 管理。`torch_whl/` 保存当前环境固定使用的本地 Torch/Torchvision wheel。

## 初始化与安装

在容器内执行：

```bash
cd /workspace/hugsim-runtime
git submodule update --init --recursive
./ensure_torch_wheels.sh
pixi install
pixi run install-apex
```

## 在 HUGSIM 中使用

在容器内激活 runtime 环境：

```bash
pixi shell --manifest-path /workspace/hugsim-runtime/pixi.toml
```

进入 HUGSIM 并暴露项目侧 `sim/` 包：

```bash
cd /workspace/HUGSIM
export PYTHONPATH=/workspace/HUGSIM/sim:$PYTHONPATH
```

验证：

```bash
python -c "import hugsim_env; import gsplat; import xformers; print('ok')"
```

## 约定

- 这个仓库不安装 `hugsim-env = { path = './sim' }`，避免共享环境绑定某个 HUGSIM 工作区。
- `apex` 不在 `pixi install` 阶段直接安装，仍通过 `pixi run install-apex` 后置安装。
- 不要手动修改 `.pixi/envs/default`，环境变更统一写入 `pixi.toml` 后重新锁定/安装。
