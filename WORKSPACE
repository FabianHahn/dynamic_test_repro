load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

HERMETIC_CC_TOOLCHAIN_VERSION = "v2.1.0"

http_archive(
    name = "hermetic_cc_toolchain",
    sha256 = "892b0dd7aa88c3504a8821e65c44fd22f32c16afab12d89e9942fff492720b37",
    urls = [
        "https://mirror.bazel.build/github.com/uber/hermetic_cc_toolchain/releases/download/{0}/hermetic_cc_toolchain-{0}.tar.gz".format(HERMETIC_CC_TOOLCHAIN_VERSION),
        "https://github.com/uber/hermetic_cc_toolchain/releases/download/{0}/hermetic_cc_toolchain-{0}.tar.gz".format(HERMETIC_CC_TOOLCHAIN_VERSION),
    ],
)

#HERMETIC_CC_TOOLCHAIN_VERSION = "v2.0.0"
#
#http_archive(
#    name = "hermetic_cc_toolchain",
#    sha256 = "57f03a6c29793e8add7bd64186fc8066d23b5ffd06fe9cc6b0b8c499914d3a65",
#    urls = [
#        "https://mirror.bazel.build/github.com/uber/hermetic_cc_toolchain/releases/download/{0}/hermetic_cc_toolchain-{0}.tar.gz".format(HERMETIC_CC_TOOLCHAIN_VERSION),
#        "https://github.com/uber/hermetic_cc_toolchain/releases/download/{0}/hermetic_cc_toolchain-{0}.tar.gz".format(HERMETIC_CC_TOOLCHAIN_VERSION),
#    ],
#)

load("@hermetic_cc_toolchain//toolchain:defs.bzl", zig_toolchains = "toolchains")

zig_toolchains()
