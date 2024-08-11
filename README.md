# Dynamic cc_test breakage repro

This repo contains a minimal reproduction case for a bug in
[hermetic_cc_toolchain](https://github.com/uber/hermetic_cc_toolchain) version v2.1.0 and
later that breaks dynamic linking of `cc_test` rules (i.e. ones that don't have `linkstatic = 1`
specified).

On v2.1.0:
```
$ bazel test ... --config=zig_linux
INFO: Invocation ID: 05e41353-4eb7-45bc-914b-95dbc0ca6249
INFO: Build option --compilation_mode has changed, discarding analysis cache.
INFO: Analyzed 3 targets (38 packages loaded, 4185 targets configured).
INFO: Found 1 target and 2 test targets...
FAIL: //:dynamic (see /home/fabian/.cache/bazel/_bazel_fabian/2cc9cc00e657a7c72ea56ec288228eca/execroot/__main__/bazel-out/k8-fastbuild/testlogs/dynamic/test.log)
INFO: Elapsed time: 33.675s, Critical Path: 17.51s
INFO: 15 processes: 4 internal, 11 linux-sandbox.
INFO: Build completed, 1 test FAILED, 15 total actions
//:static                                                                PASSED in 0.1s
//:dynamic                                                               FAILED in 0.0s
  /home/fabian/.cache/bazel/_bazel_fabian/2cc9cc00e657a7c72ea56ec288228eca/execroot/__main__/bazel-out/k8-fastbuild/testlogs/dynamic/test.log

Executed 2 out of 2 tests: 1 test passes and 1 fails locally.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option to see which ones these are.
```

Trying to print the failing test output leads to an interesting error that a shared object file cannot be found:
```
$ bazel test //:dynamic --config=zig_linux --test_output=streamed
INFO: Invocation ID: 24da0913-0973-4b94-b8f7-b978523b237f
WARNING: Streamed test output requested. All tests will be run locally, without sharding, one at a time
INFO: Analyzed target //:dynamic (24 packages loaded, 168 targets configured).
INFO: Found 1 test target...
/home/fabian/.cache/bazel/_bazel_fabian/2cc9cc00e657a7c72ea56ec288228eca/sandbox/linux-sandbox/51/execroot/__main__/bazel-out/k8-fastbuild/bin/dynamic.runfiles/__main__/dynamic: error while loading shared libraries: bazel-out/k8-fastbuild/bin/_solib_k8/liblib_Sliblib.so: cannot open shared object file: No such file or directory
FAIL: //:dynamic (see /home/fabian/.cache/bazel/_bazel_fabian/2cc9cc00e657a7c72ea56ec288228eca/execroot/__main__/bazel-out/k8-fastbuild/testlogs/dynamic/test.log)
Target //:dynamic up-to-date:
  bazel-bin/dynamic
INFO: Elapsed time: 0.371s, Critical Path: 0.20s
INFO: 2 processes: 2 linux-sandbox.
INFO: Build completed, 1 test FAILED, 2 total actions
//:dynamic                                                               FAILED in 0.0s
  /home/fabian/.cache/bazel/_bazel_fabian/2cc9cc00e657a7c72ea56ec288228eca/execroot/__main__/bazel-out/k8-fastbuild/testlogs/dynamic/test.log

Executed 1 out of 1 test: 1 fails locally.

```
The error looks the same on the latest version v3.1.0, but v2.1.0 was the first that introduced it.

To compare, this still worked fine on v2.0.0 (uncomment the commented section in `WORKSPACE`):
```
$ bazel test ... --config=zig_linux
INFO: Invocation ID: ba0d880f-2161-4b9c-9686-04066418da27
INFO: Analyzed 3 targets (4 packages loaded, 3916 targets configured).
INFO: Found 1 target and 2 test targets...
INFO: Elapsed time: 21.692s, Critical Path: 3.20s
INFO: 11 processes: 1 disk cache hit, 10 linux-sandbox.
INFO: Build completed successfully, 11 total actions
//:dynamic                                                               PASSED in 0.0s
//:static                                                                PASSED in 0.0s

Executed 2 out of 2 tests: 2 tests pass.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option to see which ones these are.
```

What is interesting is that the test binary does get built correctly and can be executed manually without problems:
```
$ bazel build //:dynamic
INFO: Invocation ID: fc5af775-9c56-4cc0-9ba2-ce2a5eea3ce8
INFO: Build options --extra_toolchains and --platforms have changed, discarding analysis cache.
INFO: Analyzed target //:dynamic (0 packages loaded, 254 targets configured).
INFO: Found 1 target...
Target //:dynamic up-to-date:
  bazel-bin/dynamic
INFO: Elapsed time: 0.640s, Critical Path: 0.47s
INFO: 8 processes: 4 internal, 4 linux-sandbox.
INFO: Build completed successfully, 8 total actions
$ bazel-bin/dynamic
The number is: 42
$ ldd bazel-bin/dynamic
	linux-vdso.so.1 (0x00007ffe41e6f000)
	liblib_Sliblib.so => /home/fabian/dev/dynamic_test_repro/bazel-bin/_solib_k8/liblib_Sliblib.so (0x00007f2ec49cc000)
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f2ec47cb000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f2ec467c000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2ec448a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f2ec49d5000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f2ec446f000)

```
