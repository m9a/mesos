# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# GENERATE PUBLIC AND INTERNAL PROTOBUFS.
#########################################
include(MesosProtobuf)

set(PUBLIC_PROTO_PATH "")
set(PUBLIC_PROTOBUF_INCLUDE_DIR "")
set(PUBLIC_PROTOBUF_SRC "")
set(INTERNAL_PROTO_PATH "")
set(INTERNAL_PROTOBUF_INCLUDE_DIR "")
set(INTERNAL_PROTOBUF_SRC "")
set(JAVA_PROTOBUF_SRC "")

# Build the protobuf structs.
#
# NOTE: The following `PROTOC_GENERATE` calls will list append to
# `PUBLIC_PROTO_PATH`, `PUBLIC_PROTOBUF_INCLUDE_DIR` and `PUBLIC_PROTOBUF_SRC`.
PROTOC_GENERATE(GRPC LIB csi TARGET csi)

# NOTE: The following `PROTOC_GENERATE` calls will list append to
# `PUBLIC_PROTOBUF_SRC`.
PROTOC_GENERATE(TARGET mesos/agent/agent)
PROTOC_GENERATE(TARGET mesos/allocator/allocator)
PROTOC_GENERATE(TARGET mesos/appc/spec)
PROTOC_GENERATE(TARGET mesos/authentication/authentication)
PROTOC_GENERATE(TARGET mesos/authorizer/acls)
PROTOC_GENERATE(TARGET mesos/authorizer/authorizer)
PROTOC_GENERATE(TARGET mesos/docker/spec)
PROTOC_GENERATE(TARGET mesos/docker/v1)
PROTOC_GENERATE(TARGET mesos/docker/v2)
PROTOC_GENERATE(TARGET mesos/maintenance/maintenance)
PROTOC_GENERATE(TARGET mesos/master/master)
PROTOC_GENERATE(TARGET mesos/module/hook)
PROTOC_GENERATE(TARGET mesos/module/module)
PROTOC_GENERATE(TARGET mesos/oci/spec)
PROTOC_GENERATE(TARGET mesos/quota/quota)
PROTOC_GENERATE(TARGET mesos/resource_provider/resource_provider)
PROTOC_GENERATE(TARGET mesos/slave/containerizer)
PROTOC_GENERATE(TARGET mesos/slave/oversubscription)
PROTOC_GENERATE(TARGET mesos/state/state)
PROTOC_GENERATE(TARGET mesos/uri/uri)

# NOTE: The following `PROTOC_GENERATE` calls will list append to
# `PUBLIC_PROTOBUF_SRC` and/or `JAVA_PROTOBUF_SRC`. The JAVA option is a noop if
# Java is disabled.
PROTOC_GENERATE(JAVA TARGET mesos/executor/executor)
PROTOC_GENERATE(JAVA TARGET mesos/fetcher/fetcher)
PROTOC_GENERATE(JAVA TARGET mesos/mesos)
PROTOC_GENERATE(JAVA TARGET mesos/scheduler/scheduler)
PROTOC_GENERATE(JAVA TARGET mesos/v1/agent/agent)
PROTOC_GENERATE(JAVA TARGET mesos/v1/allocator/allocator)
PROTOC_GENERATE(JAVA TARGET mesos/v1/executor/executor)
PROTOC_GENERATE(JAVA TARGET mesos/v1/maintenance/maintenance)
PROTOC_GENERATE(JAVA TARGET mesos/v1/master/master)
PROTOC_GENERATE(JAVA TARGET mesos/v1/mesos)
PROTOC_GENERATE(JAVA TARGET mesos/v1/quota/quota)
PROTOC_GENERATE(JAVA TARGET mesos/v1/resource_provider/resource_provider)
PROTOC_GENERATE(JAVA TARGET mesos/v1/scheduler/scheduler)

# NOTE: The following `PROTOC_GENERATE` calls will list append to
# `INTERNAL_PROTOBUF_SRC`.
PROTOC_GENERATE(INTERNAL TARGET messages/flags)
PROTOC_GENERATE(INTERNAL TARGET messages/log)
PROTOC_GENERATE(INTERNAL TARGET messages/messages)
PROTOC_GENERATE(INTERNAL TARGET messages/state)
PROTOC_GENERATE(INTERNAL TARGET slave/containerizer/mesos/isolators/network/cni/spec)
PROTOC_GENERATE(INTERNAL TARGET slave/containerizer/mesos/isolators/docker/volume/state)
PROTOC_GENERATE(INTERNAL TARGET slave/containerizer/mesos/provisioner/docker/message)
PROTOC_GENERATE(INTERNAL TARGET master/registry)
PROTOC_GENERATE(INTERNAL TARGET resource_provider/registry)
PROTOC_GENERATE(INTERNAL TARGET resource_provider/state)


# BUILD PROTOBUFS.
##################
# This must always be built as a shared library because of the following issue:
# https://github.com/google/protobuf/issues/1941
#
# Unfortunately, this is not supported on Windows. But because modules are not
# yet supported either, for now we can build it statically.
if (WIN32)
  set(MESOS_PROTOBUFS_LINKAGE STATIC)
else ()
  set(MESOS_PROTOBUFS_LINKAGE SHARED)
endif ()

add_library(
  mesos-protobufs ${MESOS_PROTOBUFS_LINKAGE}
  ${PUBLIC_PROTOBUF_SRC}
  ${INTERNAL_PROTOBUF_SRC})

target_link_libraries(mesos-protobufs PUBLIC protobuf)
target_include_directories(
  mesos-protobufs
  PUBLIC
  ${MESOS_PROTOBUF_HEADER_INCLUDE_DIRS}
  ${PUBLIC_PROTOBUF_INCLUDE_DIR}

  PRIVATE
  ${INTERNAL_PROTOBUF_INCLUDE_DIR})


# BUILD JAVA ARTIFACTS.
#######################
if (HAS_JAVA)
  add_subdirectory(java)
endif ()


# Configure Mesos files.
########################
configure_file(
  ${MESOS_PUBLIC_INCLUDE_DIR}/mesos/version.hpp.in
  ${MESOS_BIN_INCLUDE_DIR}/mesos/version.hpp)


# SOURCE FILES FOR THE MESOS LIBRARY.
#####################################
set(AGENT_SRC
  slave/compatibility.cpp
  slave/constants.cpp
  slave/container_daemon.cpp
  slave/container_logger.cpp
  slave/flags.cpp
  slave/gc.cpp
  slave/http.cpp
  slave/metrics.cpp
  slave/paths.cpp
  slave/qos_controller.cpp
  slave/qos_controllers/noop.cpp
  slave/resource_estimator.cpp
  slave/slave.cpp
  slave/state.cpp
  slave/task_status_update_manager.cpp
  slave/validation.cpp
  slave/container_loggers/sandbox.cpp
  slave/containerizer/composing.cpp
  slave/containerizer/containerizer.cpp
  slave/containerizer/docker.cpp
  slave/containerizer/fetcher.cpp
  slave/containerizer/mesos/containerizer.cpp
  slave/containerizer/mesos/isolator.cpp
  slave/containerizer/mesos/launch.cpp
  slave/containerizer/mesos/launcher.cpp
  slave/containerizer/mesos/mount.cpp
  slave/containerizer/mesos/paths.cpp
  slave/containerizer/mesos/io/switchboard.cpp
  slave/containerizer/mesos/provisioner/backend.cpp
  slave/containerizer/mesos/provisioner/paths.cpp
  slave/containerizer/mesos/provisioner/provisioner.cpp
  slave/containerizer/mesos/provisioner/store.cpp
  slave/containerizer/mesos/provisioner/appc/cache.cpp
  slave/containerizer/mesos/provisioner/appc/fetcher.cpp
  slave/containerizer/mesos/provisioner/appc/paths.cpp
  slave/containerizer/mesos/provisioner/appc/store.cpp
  slave/containerizer/mesos/provisioner/backends/copy.cpp
  slave/containerizer/mesos/provisioner/docker/local_puller.cpp
  slave/containerizer/mesos/provisioner/docker/metadata_manager.cpp
  slave/containerizer/mesos/provisioner/docker/paths.cpp
  slave/containerizer/mesos/provisioner/docker/puller.cpp
  slave/containerizer/mesos/provisioner/docker/registry_puller.cpp
  slave/containerizer/mesos/provisioner/docker/store.cpp
  slave/resource_estimators/noop.cpp)

if (WIN32)
  list(APPEND AGENT_SRC
    slave/containerizer/mesos/isolators/windows/cpu.cpp
    slave/containerizer/mesos/isolators/windows/mem.cpp)
else ()
  list(APPEND AGENT_SRC
    slave/containerizer/mesos/utils.cpp
    slave/containerizer/mesos/isolators/environment_secret.cpp
    slave/containerizer/mesos/isolators/docker/volume/driver.cpp
    slave/containerizer/mesos/isolators/docker/volume/paths.cpp
    slave/containerizer/mesos/isolators/filesystem/posix.cpp
    slave/containerizer/mesos/isolators/network/cni/paths.cpp
    slave/containerizer/mesos/isolators/network/cni/spec.cpp
    slave/containerizer/mesos/isolators/posix/disk.cpp
    slave/containerizer/mesos/isolators/posix/rlimits.cpp
    slave/containerizer/mesos/isolators/volume/sandbox_path.cpp
    slave/containerizer/mesos/provisioner/utils.cpp)
endif ()

set(APPC_SRC
  appc/spec.cpp)

set(AUTHENTICATION_SRC
  authentication/cram_md5/authenticatee.cpp
  authentication/cram_md5/authenticator.cpp
  authentication/cram_md5/auxprop.cpp
  authentication/http/basic_authenticatee.cpp
  authentication/http/basic_authenticator_factory.cpp
  authentication/http/combined_authenticator.cpp)

if (ENABLE_SSL)
  list(APPEND AUTHENTICATION_SRC
    authentication/executor/jwt_secret_generator.cpp)
endif ()

set(AUTHORIZER_SRC
  authorizer/acls.cpp
  authorizer/authorizer.cpp
  authorizer/local/authorizer.cpp)

set(COMMON_SRC
  common/attributes.cpp
  common/build.cpp
  common/command_utils.cpp
  common/http.cpp
  common/protobuf_utils.cpp
  common/resources.cpp
  common/resources_utils.cpp
  common/roles.cpp
  common/type_utils.cpp
  common/validation.cpp
  common/values.cpp)

set(DOCKER_SRC
  docker/docker.cpp
  docker/spec.cpp)

set(EXECUTOR_SRC
  exec/exec.cpp
  executor/executor.cpp
  executor/v0_v1executor.cpp)

set(FILES_SRC
  files/files.cpp)

set(HDFS_SRC
  hdfs/hdfs.cpp)

set(HEALTH_CHECK_SRC
  checks/checker.cpp
  checks/checker_process.cpp
  checks/health_checker.cpp)

set(INTERNAL_SRC
  internal/devolve.cpp
  internal/evolve.cpp)

set(LINUX_SRC
  linux/capabilities.cpp
  linux/cgroups.cpp
  linux/fs.cpp
  linux/ldcache.cpp
  linux/ldd.cpp
  linux/ns.cpp
  linux/perf.cpp
  linux/systemd.cpp
  slave/containerizer/mesos/linux_launcher.cpp
  slave/containerizer/mesos/isolators/appc/runtime.cpp
  slave/containerizer/mesos/isolators/cgroups/cgroups.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystem.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/blkio.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/cpu.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/cpuacct.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/cpuset.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/devices.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/hugetlb.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/memory.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/net_cls.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/net_prio.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/perf_event.cpp
  slave/containerizer/mesos/isolators/cgroups/subsystems/pids.cpp
  slave/containerizer/mesos/isolators/docker/runtime.cpp
  slave/containerizer/mesos/isolators/docker/volume/isolator.cpp
  slave/containerizer/mesos/isolators/filesystem/linux.cpp
  slave/containerizer/mesos/isolators/filesystem/shared.cpp
  slave/containerizer/mesos/isolators/gpu/allocator.cpp
  slave/containerizer/mesos/isolators/gpu/isolator.cpp
  slave/containerizer/mesos/isolators/gpu/nvml.cpp
  slave/containerizer/mesos/isolators/gpu/volume.cpp
  slave/containerizer/mesos/isolators/linux/capabilities.cpp
  slave/containerizer/mesos/isolators/namespaces/ipc.cpp
  slave/containerizer/mesos/isolators/namespaces/pid.cpp
  slave/containerizer/mesos/isolators/network/cni/cni.cpp
  slave/containerizer/mesos/isolators/volume/host_path.cpp
  slave/containerizer/mesos/isolators/volume/image.cpp
  slave/containerizer/mesos/isolators/volume/secret.cpp
  slave/containerizer/mesos/provisioner/backends/aufs.cpp
  slave/containerizer/mesos/provisioner/backends/bind.cpp
  slave/containerizer/mesos/provisioner/backends/overlay.cpp)

set(LOCAL_SRC
  local/local.cpp)

set(LOG_SRC
  log/catchup.cpp
  log/consensus.cpp
  log/coordinator.cpp
  log/leveldb.cpp
  log/log.cpp
  log/main.cpp
  log/metrics.cpp
  log/recover.cpp
  log/replica.cpp
  log/tool/benchmark.cpp
  log/tool/initialize.cpp
  log/tool/read.cpp
  log/tool/replica.cpp)

set(LOGGING_SRC
  logging/flags.cpp
  logging/logging.cpp)

set(MASTER_SRC
  master/constants.cpp
  master/flags.cpp
  master/http.cpp
  master/maintenance.cpp
  master/master.cpp
  master/metrics.cpp
  master/quota.cpp
  master/quota_handler.cpp
  master/registrar.cpp
  master/registry_operations.cpp
  master/weights.cpp
  master/weights_handler.cpp
  master/validation.cpp
  master/allocator/allocator.cpp
  master/allocator/mesos/hierarchical.cpp
  master/allocator/mesos/metrics.cpp
  master/allocator/sorter/drf/metrics.cpp
  master/allocator/sorter/drf/sorter.cpp
  master/contender/contender.cpp
  master/contender/standalone.cpp
  master/contender/zookeeper.cpp
  master/detector/detector.cpp
  master/detector/standalone.cpp
  master/detector/zookeeper.cpp)

set(MESSAGES_SRC
  messages/messages.cpp)

set(MODULE_SRC
  hook/manager.cpp
  module/manager.cpp)

set(OCI_SRC
  oci/spec.cpp)

set(POSIX_SRC
  posix/rlimits.cpp)

set(RESOURCE_PROVIDER_SRC
  resource_provider/daemon.cpp
  resource_provider/detector.cpp
  resource_provider/driver.cpp
  resource_provider/local.cpp
  resource_provider/manager.cpp
  resource_provider/registrar.cpp
  resource_provider/validation.cpp)

set(SCHEDULER_SRC
  sched/sched.cpp
  scheduler/scheduler.cpp)

set(SECRET_SRC
  secret/resolver.cpp)

set(STATE_SRC
  state/in_memory.cpp)

set(STATUS_UPDATE_MANAGER_SRC
  status_update_manager/operation.cpp)

if (NOT WIN32)
  list(APPEND STATE_SRC
    state/leveldb.cpp
    state/log.cpp
    state/zookeeper.cpp)
endif ()

set(URI_SRC
  uri/fetcher.cpp
  uri/utils.cpp
  uri/fetchers/copy.cpp
  uri/fetchers/curl.cpp
  uri/fetchers/hadoop.cpp)

if (NOT WIN32)
  list(APPEND URI_SRC
    uri/fetchers/docker.cpp)
endif ()

set(USAGE_SRC
  usage/usage.cpp)

set(V1_SRC
  v1/attributes.cpp
  v1/mesos.cpp
  v1/resources.cpp
  v1/values.cpp)

set(VERSION_SRC
  version/version.cpp)

set(WATCHER_SRC
  watcher/whitelist_watcher.cpp)

set(WIN32_SRC
  slave/containerizer/mesos/isolators/environment_secret.cpp
  slave/containerizer/mesos/isolators/filesystem/posix.cpp
  slave/containerizer/mesos/isolators/filesystem/windows.cpp)

set(ZOOKEEPER_SRC
  zookeeper/authentication.cpp
  zookeeper/contender.cpp
  zookeeper/detector.cpp
  zookeeper/zookeeper.cpp
  zookeeper/group.cpp)

set(MESOS_SRC
  ${AGENT_SRC}
  ${APPC_SRC}
  ${AUTHENTICATION_SRC}
  ${AUTHORIZER_SRC}
  ${COMMON_SRC}
  ${DOCKER_SRC}
  ${EXECUTOR_SRC}
  ${FILES_SRC}
  ${HDFS_SRC}
  ${HEALTH_CHECK_SRC}
  ${INTERNAL_SRC}
  ${LOCAL_SRC}
  ${LOGGING_SRC}
  ${MASTER_SRC}
  ${MESSAGES_SRC}
  ${MODULE_SRC}
  ${OCI_SRC}
  ${RESOURCE_PROVIDER_SRC}
  ${SCHEDULER_SRC}
  ${SECRET_SRC}
  ${STATE_SRC}
  ${STATUS_UPDATE_MANAGER_SRC}
  ${URI_SRC}
  ${USAGE_SRC}
  ${V1_SRC}
  ${VERSION_SRC}
  ${WATCHER_SRC}
  ${ZOOKEEPER_SRC})

if (NOT WIN32)
  list(APPEND MESOS_SRC
    ${LOG_SRC}
    ${POSIX_SRC})
endif ()

# Include source for linux build.
if (LINUX)
  list(APPEND MESOS_SRC ${LINUX_SRC})
endif ()

# Include source for Windows build.
if (WIN32)
  list(APPEND MESOS_SRC ${WIN32_SRC})
endif ()

# Generate all binaries in the same folder.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/src)
if (WIN32)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/src)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/src)
endif ()

# Generate all libraries in the same folder to conform with the Automake build.
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/src/.libs)
if (WIN32)
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/src/.libs)
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/src/.libs)
endif ()


# THE MESOS LIBRARY
###################
add_library(mesos ${MESOS_SRC})

set_target_properties(
  mesos PROPERTIES
  VERSION ${MESOS_PACKAGE_VERSION}
  SOVERSION ${MESOS_PACKAGE_SOVERSION})

target_compile_definitions(mesos PUBLIC USE_CMAKE_BUILD_CONFIG)

target_include_directories(
  mesos PUBLIC
  ${MESOS_PUBLIC_INCLUDE_DIR}
  ${MESOS_BIN_INCLUDE_DIR}
  ${MESOS_BIN_SRC_DIR}
  ${MESOS_SRC_DIR})

target_link_libraries(
  mesos PUBLIC
  process
  sasl2
  zookeeper
  mesos-protobufs
  $<$<PLATFORM_ID:Linux>:nvml>
  $<$<BOOL:${HAS_JAVA}>:mesos-java>)

if (NOT WIN32)
  target_link_libraries(mesos PUBLIC leveldb)
endif ()

if (ENABLE_PRECOMPILED_HEADERS)
  set_target_properties(
    mesos PROPERTIES
    COTIRE_ADD_UNITY_BUILD FALSE
    COTIRE_PREFIX_HEADER_INCLUDE_PATH "${CMAKE_SOURCE_DIR}/3rdparty;${CMAKE_BINARY_DIR}/3rdparty")

  cotire(mesos)
endif ()


# BUILD THE MESOS EXECUTABLES.
##############################
add_subdirectory(checks)
add_subdirectory(cli)
add_subdirectory(docker)
add_subdirectory(examples)
add_subdirectory(launcher)
add_subdirectory(local)
add_subdirectory(log)
add_subdirectory(master)
add_subdirectory(slave)
add_subdirectory(slave/containerizer/mesos)
add_subdirectory(usage)


# BUILD THE MESOS TESTS.
########################
add_subdirectory(tests)
