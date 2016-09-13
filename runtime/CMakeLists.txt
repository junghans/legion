#------------------------------------------------------------------------------#
# Copyright 2016 Kitware, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#------------------------------------------------------------------------------#

# Install headers
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/legion
  DESTINATION include
  FILES_MATCHING PATTERN "*/*.h" PATTERN "*/*.inl"
)

# Lowlevel runtime
if(Legion_USE_SHARED_LOWLEVEL)
  list(APPEND LOW_RUNTIME_SRC shared_lowlevel.cc)
else()
  list(APPEND LOW_RUNTIME_SRC
    legion/lowlevel/lowlevel_config.h
    legion/lowlevel/lowlevel_disk.cc
    legion/lowlevel/lowlevel_dma.h   legion/lowlevel/lowlevel_dma.cc
    legion/lowlevel/lowlevel_dma.inl
    legion/lowlevel/lowlevel.h       legion/lowlevel/lowlevel.cc
    legion/lowlevel/lowlevel_impl.h
    legion/realm/event_impl.h        legion/realm/event_impl.cc
    legion/realm/event_impl.inl
    legion/realm/faults.h            legion/realm/faults.cc
    legion/realm/faults.inl
    legion/realm/idx_impl.h          legion/realm/idx_impl.cc
    legion/realm/inst_impl.h         legion/realm/inst_impl.cc
    legion/realm/machine_impl.h      legion/realm/machine_impl.cc
    legion/realm/mem_impl.h          legion/realm/mem_impl.cc
    legion/realm/metadata.h          legion/realm/metadata.cc
    legion/realm/module.h            legion/realm/module.cc
    legion/realm/nodeset.h
    legion/realm/numa/numa_module.h  legion/realm/numa/numa_module.cc
    legion/realm/numa/numasysif.h    legion/realm/numa/numasysif.cc
    legion/realm/operation.h         legion/realm/operation.cc
    legion/realm/operation.inl
    legion/realm/proc_impl.h         legion/realm/proc_impl.cc
    legion/realm/procset/procset_module.h legion/realm/procset/procset_module.cc
    legion/realm/rsrv_impl.h         legion/realm/rsrv_impl.cc
    legion/realm/runtime_impl.h      legion/realm/runtime_impl.cc
    legion/realm/sampling_impl.h     legion/realm/sampling_impl.cc
    legion/realm/tasks.h             legion/realm/tasks.cc
    legion/realm/threads.h           legion/realm/threads.cc
    legion/realm/threads.inl
  )
  if(Legion_USE_CUDA)
    list(APPEND LOW_RUNTIME_SRC
      legion/realm/cuda/cuda_module.h    legion/realm/cuda/cuda_module.cc
      legion/realm/cuda/cudart_hijack.h  legion/realm/cuda/cudart_hijack.cc
    )
  endif()
  if(Legion_USE_LLVM)
    list(APPEND LOW_RUNTIME_SRC
      legion/realm/llvmjit/llvmjit.h
      legion/realm/llvmjit/llvmjit.inl
      legion/realm/llvmjit/llvmjit_internal.h  legion/realm/llvmjit/llvmjit_internal.cc
      legion/realm/llvmjit/llvmjit_module.h    legion/realm/llvmjit/llvmjit_module.cc
  )
  endif()
  if(Legion_USE_GASNet)
    list(APPEND LOW_RUNTIME_SRC activemsg.h activemsg.cc)
  endif()
endif()
list(APPEND LOW_RUNTIME_SRC
  legion/lowlevel/accessor.h
  legion/lowlevel/arrays.h
  legion/lowlevel/atomics.h
  legion/lowlevel/common.h
  legion/realm/bytearray.h
  legion/realm/bytearray.inl
  legion/realm/circ_queue.h
  legion/realm/circ_queue.inl
  legion/realm/cmdline.h          legion/realm/cmdline.cc
  legion/realm/cmdline.inl
  legion/realm/codedesc.h         legion/realm/codedesc.cc
  legion/realm/codedesc.inl
  legion/realm/custom_serdez.h
  legion/realm/custom_serdez.inl
  legion/realm/dynamic_table.h
  legion/realm/dynamic_table.inl
  legion/realm/event.h
  legion/realm/id.h
  legion/realm/id.inl
  legion/realm/indexspace.h
  legion/realm/instance.h
  legion/realm/logging.h          legion/realm/logging.cc
  legion/realm/logging.inl
  legion/realm/machine.h
  legion/realm/machine.inl
  legion/realm/memory.h
  legion/realm/pri_queue.h
  legion/realm/pri_queue.inl
  legion/realm/processor.h
  legion/realm/processor.inl
  legion/realm/profiling.h        legion/realm/profiling.cc
  legion/realm/profiling.inl
  legion/realm/realm_config.h
  legion/realm/realm.h
  legion/realm/redop.h
  legion/realm/reservation.h
  legion/realm/runtime.h
  legion/realm/sampling.h
  legion/realm/sampling.inl
  legion/realm/serialize.h
  legion/realm/serialize.inl
  legion/realm/timers.h           legion/realm/timers.cc
  legion/realm/timers.inl
  legion/realm/utils.h
  legion/lowlevel/utilities.h
)
find_package(Threads REQUIRED)
add_library(LowLevelRuntime ${LOW_RUNTIME_SRC})
target_link_libraries(LowLevelRuntime
  PRIVATE ${CMAKE_DL_LIBS} ${CMAKE_THREAD_LIBS_INIT}
)
set_target_properties(LowLevelRuntime PROPERTIES POSITION_INDEPENDENT_CODE ON)
set_target_properties(LowLevelRuntime PROPERTIES SOVERSION ${SOVERSION})
target_compile_definitions(LowLevelRuntime
  PRIVATE
    $<$<OR:$<CONFIG:Debug>,$<CONFIG:RelWithDebInfo>>:DEBUG_LOW_LEVEL>
  PUBLIC
    COMPILE_TIME_MIN_LEVEL=LEVEL_${Legion_OUTPUT_LEVEL}
)

if(Legion_USE_HWLOC)
  target_compile_definitions(LowLevelRuntime PRIVATE REALM_USE_HWLOC)
  target_link_libraries(LowLevelRuntime PRIVATE HWLOC::HWLOC)
endif()

if(Legion_USE_GASNet)
  target_compile_definitions(LowLevelRuntime PUBLIC USE_GASNET)
  target_link_libraries(LowLevelRuntime PUBLIC GASNet::GASNet)
endif()

if(Legion_USE_LLVM)
  target_compile_definitions(LowLevelRuntime PRIVATE REALM_USE_LLVM)
  target_link_libraries(LowLevelRuntime PRIVATE LLVM::LLVM)
endif()

if(Legion_USE_CUDA)
  target_compile_definitions(LowLevelRuntime PUBLIC USE_CUDA)
  target_include_directories(LowLevelRuntime PRIVATE ${CUDA_INCLUDE_DIRS})
  target_link_libraries(LowLevelRuntime PRIVATE ${CUDA_CUDA_LIBRARY})
endif()

if(Legion_USE_SHARED_LOWLEVEL)
  set_target_properties(LowLevelRuntime PROPERTIES OUTPUT_NAME sharedllr)
else()
  set_target_properties(LowLevelRuntime PROPERTIES OUTPUT_NAME realm)
endif()

target_include_directories(LowLevelRuntime
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/legion>
    $<INSTALL_INTERFACE:include/legion>
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} lowlevel realm
)

install(TARGETS LowLevelRuntime EXPORT LegionTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

# Mapper objects
list(APPEND MAPPER_SRC
  legion/mappers/debug_mapper.h       legion/mappers/debug_mapper.cc
  legion/mappers/default_mapper.h     legion/mappers/default_mapper.cc
  legion/mappers/mapping_utilities.h  legion/mappers/mapping_utilities.cc
  legion/mappers/replay_mapper.h      legion/mappers/replay_mapper.cc
  legion/mappers/shim_mapper.h        legion/mappers/shim_mapper.cc
  legion/mappers/test_mapper.h        legion/mappers/test_mapper.cc
)

# High level runtime
list(APPEND HIGH_RUNTIME_SRC
  legion/field_tree.h
  legion/garbage_collection.h  legion/garbage_collection.cc
  legion/interval_tree.h
  legion/legion_allocation.h
  legion/legion_analysis.h     legion/legion_analysis.cc
  legion/legion_c.h            legion/legion_c.cc
  legion/legion_config.h
  legion/legion_constraint.h   legion/legion_constraint.cc
  legion/legion_c_util.h
  legion/legion.h              legion/legion.cc
  legion/legion.inl
  legion/legion_instances.h    legion/legion_instances.cc
  legion/legion_mapping.h      legion/legion_mapping.cc
  legion/legion_ops.h          legion/legion_ops.cc
  legion/legion_profiling.h    legion/legion_profiling.cc
  legion/legion_realm.h
  legion/legion_spy.h          legion/legion_spy.cc
  legion/legion_tasks.h        legion/legion_tasks.cc
  legion/legion_trace.h        legion/legion_trace.cc
  legion/legion_types.h
  legion/legion_utilities.h
  legion/legion_views.h        legion/legion_views.cc
  legion/mapper_manager.h      legion/mapper_manager.cc
  legion/rectangle_set.h
  legion/region_tree.h         legion/region_tree.cc
  legion/runtime.h             legion/runtime.cc
)

add_library(HighLevelRuntime ${MAPPER_SRC} ${HIGH_RUNTIME_SRC})
target_link_libraries(HighLevelRuntime LowLevelRuntime)
set_target_properties(HighLevelRuntime PROPERTIES POSITION_INDEPENDENT_CODE ON)
set_target_properties(HighLevelRuntime PROPERTIES OUTPUT_NAME legion)
set_target_properties(HighLevelRuntime PROPERTIES SOVERSION ${SOVERSION})
target_compile_definitions(HighLevelRuntime
  PRIVATE
    ASSUME_UNALLOCABLE
    $<$<OR:$<CONFIG:Debug>,$<CONFIG:RelWithDebInfo>>:DEBUG_HIGH_LEVEL>
  PUBLIC
    MAX_FIELDS=${Legion_MAX_FIELDS}
)
target_include_directories(HighLevelRuntime
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/legion>
    $<INSTALL_INTERFACE:include/legion>
  PRIVATE lowlevel realm mappers
)

install(TARGETS HighLevelRuntime EXPORT LegionTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

add_library(Legion INTERFACE)
set_target_properties(Legion PROPERTIES
  INTERFACE_LINK_LIBRARIES HighLevelRuntime
)
install(TARGETS Legion EXPORT LegionTargets)
