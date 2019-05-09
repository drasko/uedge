# uEdge
Mainflux micro-edge seervice.

## Install

```
git clone git@github.com:drasko/uedge.git
```

### Compile

#### OpenWrt
Follow the instructions [here](https://github.com/japaric/rust-cross) to add `mips-unknown-linux-musl` target.

Additionally, in `~/.cargo/config` add OpenWrt target library path (obtained after OpenWrt buld)
to `rustflags` so that you can have a setup like this:

```
[target.mips-unknown-linux-musl]
linker = "mips-openwrt-linux-musl-gcc"
rustflags = ["-L", "/home/drasko/openwrt/staging_dir/target-mips_24kc_musl/usr/lib"]
```

uEdge can then be cross-compiled for MIMPS-based OpenWrt system:

```
OPENSSL_SEARCH_PATH=<path_to_openssl_lib> CC=mips-openwrt-linux-musl-gcc STAGING_DIR=<path_to_openwrt_staging_dir> cargo build --target mips-unknown-linux-musl --release
```

Example:

```
OPENSSL_SEARCH_PATH=/home/drasko/openwrt/staging_dir/target-mips_24kc_musl/usr CC=mips-openwrt-linux-musl-gcc STAGING_DIR=/home/drasko/openwrt/staging_dir cargo build --target mips-unknown-linux-musl --release
```

> N.B. - For this to work, [PR](https://github.com/eclipse/paho.mqtt.rust/pull/40) that
> enables Paho MQTT static build and OpenSSL lib finding must be applied.
>

Also, note that if dynamic linking is needed, than `paho.mqtt.rust/paho-mqtt-sys/build.rs` should be
```
if cfg!(feature = "ssl") {
  println!("cargo:rustc-link-lib=ssl");
  println!("cargo:rustc-link-lib=crypto");
}
```

But then you must install `libssl` and `libcrypto` in OpenWrt system (which will probably be the case because of Mosquitto).

Dynamically linked binary can be very small:
```
drasko@Marx:~/rust/uedge$ ls -la target/mips-unknown-linux-musl/release/uedge
-rwxr-xr-x 1 drasko drasko 314564 May  9 01:11 target/mips-unknown-linux-musl/release/uedge
```

For static linking of `libssl` and `libcrypto` use:
Also, note that if dynamic linking is needed, than `paho.mqtt.rust/paho-mqtt-sys/build.rs` should be
```
if cfg!(feature = "ssl") {
  println!("cargo:rustc-link-lib=static=ssl");
  println!("cargo:rustc-link-lib=static=crypto");
}
```

#### Deploy
Binary can be further optimized by stripping:
```
mips-openwrt-linux-strip target/mips-unknown-linux-musl/release/uedge
```

Then it can be bruttaly compressed with [upx](https://upx.github.io/):
```
upx --ultra-brute target/mips-unknown-linux-musl/release/uedge
```

Copy binary to the board:
```
scp target/mips-unknown-linux-musl/release/uedge root@lima.local:/tmp
```

Execute:
```
RUST_LOG=debug /tmp/uedge
```

## License
[Apache-2.0](LICENSE)