### Rust 编译慢/卡死的问题

#### 1. 卡在依赖库

如果是在编译依赖库的过程中卡死，如：

![](C:\UserFile\四下\compiling.png)

那有可能是在找编译库的时候网络卡死了。

Rust 的包管理器 `cargo` 会自动下载依赖库，存入本地的 `~/.cargo/` 文件夹（类似 Python 的 `pip`），并在每次编译时检查更新。一般来说，访问Rust官方镜像 `crates.io` 上的库还是没问题的，但如果在项目的 `Cargo.toml` 中记录的依赖里特殊指定了 git 路径，则会去对应 git 仓库检查更新，**特别是当 git 路径指向 github 时，使用国内网络直连容易出问题**，如 `ch2` 仓库的

```rust
[dependencies]
riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }

```

##### 简单解决办法

###### 1. 配置代理/换源

在这里不讨论，可以参考文档  https://learningos.github.io/rCore-Tutorial-Guide-2023S/0setup-devel-env.html

###### 2. 使用 `offline` 参数

如果已经成功下载过一次，可以在编译时加上 `--offline` 参数，让 `cargo` 尝试从本地而不`Cargo.toml` 指定的 URL 拉取项目。

以 `ch2`为例，在 `os/Makefile` 的：

```makefile
kernel:
	@cd ../user && make build
	@echo Platform: $(BOARD)
	@cp src/linker-$(BOARD).ld src/linker.ld
	@cargo build $(MODE_ARG)
	@rm src/linker.ld
```

其中 `cargo build $(MODE_ARG)` 一行编译了内核。将其修改成 `cargo build --offline $(MODE_ARG)` 即可。

#### 2. 卡在 Blocking

如果在 `cargo build`编译后显示类似

```shell
    Blocking waiting for file lock on build directory
```

的消息，可以**删除 `~/.cargo/.package-cache` 后再编译**。比如：

```shell
rm ~/.cargo/.package-cache
```


