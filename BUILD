package(default_visibility = ["//visibility:public"])

cc_test(
    name = "dynamic",
    srcs = [
        "test.cpp",
    ],
    deps = [
        "//lib",
    ]
)

cc_test(
    name = "static",
    srcs = [
        "test.cpp",
    ],
    deps = [
        "//lib",
    ],
    linkstatic = 1
)
