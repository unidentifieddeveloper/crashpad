cmake_minimum_required(VERSION 3.14)

project(padcr)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

if(MSVC)
    include(MSVCRuntime)

    if(NOT DEFINED MSVC_RUNTIME)
        set(MSVC_RUNTIME "static")
    endif()

    configure_msvc_runtime()
    add_compile_options(/wd4530)
endif()

enable_language(ASM_MASM)

add_library(
    mini_chromium
    STATIC
    vendor/mini_chromium/base/logging
    vendor/mini_chromium/base/rand_util
    vendor/mini_chromium/base/files/file_path
    vendor/mini_chromium/base/files/scoped_file
    vendor/mini_chromium/base/process/memory
    vendor/mini_chromium/base/process/process_metrics_win
    vendor/mini_chromium/base/strings/stringprintf
    vendor/mini_chromium/base/strings/string_number_conversions
    vendor/mini_chromium/base/strings/string_util_win
    vendor/mini_chromium/base/strings/utf_string_conversion_utils
    vendor/mini_chromium/base/strings/utf_string_conversions
    vendor/mini_chromium/base/synchronization/lock
    vendor/mini_chromium/base/synchronization/lock_impl_win
    vendor/mini_chromium/base/third_party/icu/icu_utf
)

target_compile_definitions(
    mini_chromium
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    mini_chromium
    PUBLIC
    vendor/mini_chromium
)

add_library(
    getopt
    STATIC
    vendor/crashpad/third_party/getopt/getopt
)

target_compile_definitions(
    getopt
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    getopt
    PUBLIC
    vendor/crashpad/third_party/getopt
)

add_library(
    crashpad_compat
    STATIC
    vendor/crashpad/compat/win/strings.cc
    vendor/crashpad/compat/win/time.cc
    vendor/crashpad/compat/win/time.h
)

target_compile_definitions(
    crashpad_compat
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    crashpad_compat
    PUBLIC
    vendor/crashpad/compat/win
)

add_library(
    crashpad_tools
    STATIC
    vendor/crashpad/tools/tool_support
)

target_compile_definitions(
    crashpad_tools
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    crashpad_tools
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_tools
    PUBLIC
    mini_chromium
)

add_library(
    crashpad_util
    STATIC
    vendor/crashpad/util/file/file_io
    vendor/crashpad/util/file/file_io_win
    vendor/crashpad/util/file/file_reader
    vendor/crashpad/util/file/file_seeker
    vendor/crashpad/util/file/file_writer
    vendor/crashpad/util/file/filesystem_win
    vendor/crashpad/util/file/scoped_remove_file
    vendor/crashpad/util/misc/capture_context_win.asm
    vendor/crashpad/util/misc/initialization_state_dcheck
    vendor/crashpad/util/misc/metrics
    vendor/crashpad/util/misc/paths_win
    vendor/crashpad/util/misc/random_string
    vendor/crashpad/util/misc/time_win
    vendor/crashpad/util/misc/uuid
    vendor/crashpad/util/misc/zlib
    vendor/crashpad/util/net/http_body
    vendor/crashpad/util/net/http_body_gzip
    vendor/crashpad/util/net/http_multipart_builder
    vendor/crashpad/util/net/http_transport
    vendor/crashpad/util/net/http_transport_win
    vendor/crashpad/util/net/url
    vendor/crashpad/util/numeric/checked_address_range
    vendor/crashpad/util/process/process_memory
    vendor/crashpad/util/process/process_memory_range
    vendor/crashpad/util/process/process_memory_win
    vendor/crashpad/util/stdlib/aligned_allocator
    vendor/crashpad/util/stdlib/string_number_conversion
    vendor/crashpad/util/stdlib/strlcpy
    vendor/crashpad/util/string/split_string
    vendor/crashpad/util/synchronization/semaphore_win
    vendor/crashpad/util/thread/thread
    vendor/crashpad/util/thread/thread_win
    vendor/crashpad/util/thread/worker_thread
    vendor/crashpad/util/win/command_line
    vendor/crashpad/util/win/critical_section_with_debug_info
    vendor/crashpad/util/win/exception_handler_server
    vendor/crashpad/util/win/get_function
    vendor/crashpad/util/win/handle
    vendor/crashpad/util/win/initial_client_data
    vendor/crashpad/util/win/module_version
    vendor/crashpad/util/win/nt_internals
    vendor/crashpad/util/win/ntstatus_logging
    vendor/crashpad/util/win/process_info
    vendor/crashpad/util/win/registration_protocol_win
    vendor/crashpad/util/win/safe_terminate_process.asm
    vendor/crashpad/util/win/session_end_watcher
    vendor/crashpad/util/win/scoped_handle
    vendor/crashpad/util/win/scoped_process_suspend
    vendor/crashpad/util/win/scoped_set_event
)

target_compile_definitions(
    crashpad_util
    PRIVATE
    -DCRASHPAD_ZLIB_SOURCE_EXTERNAL
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

if(NOT DEFINED ZLIB_LIBRARY)
    message(WARNING "ZLIB_LIBRARY not set")
endif()

if(NOT DEFINED ZLIB_INCLUDE_DIR)
    message(WARNING "ZLIB_INCLUDE_DIR not set")
endif()

target_link_libraries(
    crashpad_util
    PRIVATE
    mini_chromium
    crashpad_compat
    ${ZLIB_LIBRARY}
    user32
    version
    winhttp
)

target_include_directories(
    crashpad_util
    PUBLIC
    vendor/crashpad
    ${ZLIB_INCLUDE_DIR}
)

add_library(
    crashpad_client
    STATIC
    vendor/crashpad/client/annotation
    vendor/crashpad/client/annotation_list
    vendor/crashpad/client/crash_report_database
    vendor/crashpad/client/crash_report_database_win
    vendor/crashpad/client/crashpad_client_win
    vendor/crashpad/client/crashpad_info
    vendor/crashpad/client/prune_crash_reports
    vendor/crashpad/client/settings
)

target_compile_definitions(
    crashpad_client
    PRIVATE
    -DNOMINMAX
    -DUNICODE
)

target_include_directories(
    crashpad_client
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_client
    PRIVATE
    mini_chromium
    crashpad_compat
    crashpad_util
)

add_library(
    crashpad_minidump
    STATIC
    vendor/crashpad/minidump/minidump_annotation_writer
    vendor/crashpad/minidump/minidump_byte_array_writer
    vendor/crashpad/minidump/minidump_context_writer
    vendor/crashpad/minidump/minidump_crashpad_info_writer
    vendor/crashpad/minidump/minidump_exception_writer
    vendor/crashpad/minidump/minidump_file_writer
    vendor/crashpad/minidump/minidump_handle_writer
    vendor/crashpad/minidump/minidump_memory_writer
    vendor/crashpad/minidump/minidump_memory_info_writer
    vendor/crashpad/minidump/minidump_misc_info_writer
    vendor/crashpad/minidump/minidump_module_crashpad_info_writer
    vendor/crashpad/minidump/minidump_module_writer
    vendor/crashpad/minidump/minidump_rva_list_writer
    vendor/crashpad/minidump/minidump_simple_string_dictionary_writer
    vendor/crashpad/minidump/minidump_stream_writer
    vendor/crashpad/minidump/minidump_string_writer
    vendor/crashpad/minidump/minidump_system_info_writer
    vendor/crashpad/minidump/minidump_thread_id_map
    vendor/crashpad/minidump/minidump_thread_writer
    vendor/crashpad/minidump/minidump_unloaded_module_writer
    vendor/crashpad/minidump/minidump_user_stream_writer
    vendor/crashpad/minidump/minidump_writable
    vendor/crashpad/minidump/minidump_writer_util
)

target_compile_definitions(
    crashpad_minidump
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    crashpad_minidump
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_minidump
    PRIVATE
    mini_chromium
    crashpad_compat
    crashpad_util
)

add_library(
    crashpad_snapshot
    STATIC
    vendor/crashpad/snapshot/annotation_snapshot
    vendor/crashpad/snapshot/capture_memory
    vendor/crashpad/snapshot/cpu_context
    vendor/crashpad/snapshot/crashpad_info_client_options
    vendor/crashpad/snapshot/handle_snapshot
    vendor/crashpad/snapshot/memory_snapshot
    vendor/crashpad/snapshot/unloaded_module_snapshot
    vendor/crashpad/snapshot/crashpad_types/crashpad_info_reader
    vendor/crashpad/snapshot/minidump/exception_snapshot_minidump
    vendor/crashpad/snapshot/minidump/memory_snapshot_minidump
    vendor/crashpad/snapshot/minidump/minidump_annotation_reader
    vendor/crashpad/snapshot/minidump/minidump_context_converter
    vendor/crashpad/snapshot/minidump/minidump_simple_string_dictionary_reader
    vendor/crashpad/snapshot/minidump/minidump_string_list_reader
    vendor/crashpad/snapshot/minidump/minidump_string_reader
    vendor/crashpad/snapshot/minidump/module_snapshot_minidump
    vendor/crashpad/snapshot/minidump/process_snapshot_minidump
    vendor/crashpad/snapshot/minidump/system_snapshot_minidump
    vendor/crashpad/snapshot/minidump/thread_snapshot_minidump
    vendor/crashpad/snapshot/win/capture_memory_delegate_win
    vendor/crashpad/snapshot/win/cpu_context_win
    vendor/crashpad/snapshot/win/exception_snapshot_win
    vendor/crashpad/snapshot/win/memory_map_region_snapshot_win
    vendor/crashpad/snapshot/win/module_snapshot_win
    vendor/crashpad/snapshot/win/pe_image_annotations_reader
    vendor/crashpad/snapshot/win/pe_image_reader
    vendor/crashpad/snapshot/win/pe_image_resource_reader
    vendor/crashpad/snapshot/win/process_reader_win
    vendor/crashpad/snapshot/win/process_snapshot_win
    vendor/crashpad/snapshot/win/process_subrange_reader
    vendor/crashpad/snapshot/win/system_snapshot_win
    vendor/crashpad/snapshot/win/thread_snapshot_win
)

target_compile_definitions(
    crashpad_snapshot
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    crashpad_snapshot
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_snapshot
    PRIVATE
    mini_chromium
    crashpad_compat
    powrprof
)

add_executable(
    crashpad_handler
    WIN32
    vendor/crashpad/handler/crash_report_upload_thread
    vendor/crashpad/handler/handler_main
    vendor/crashpad/handler/main
    vendor/crashpad/handler/minidump_to_upload_parameters
    vendor/crashpad/handler/prune_crash_reports_thread
    vendor/crashpad/handler/user_stream_data_source
    vendor/crashpad/handler/win/crash_report_exception_handler
)

target_compile_definitions(
    crashpad_handler
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    crashpad_handler
    PRIVATE
    vendor/crashpad
    vendor/crashpad/compat/win
    vendor/mini_chromium
)

target_link_libraries(
    crashpad_handler
    PRIVATE
    crashpad_client
    crashpad_minidump
    crashpad_snapshot
    crashpad_tools
    getopt
)

# Make a .com file copy from the .exe with subsystem console
add_custom_command(
    TARGET crashpad_handler POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:crashpad_handler>
        "$<TARGET_FILE_DIR:crashpad_handler>/crashpad_handler.com"
)

add_custom_command(
    TARGET crashpad_handler POST_BUILD
    COMMAND editbin
        -nologo
        -subsystem:console
        "$<TARGET_FILE_DIR:crashpad_handler>/crashpad_handler.com"
)

add_executable(
    generate_dump
        vendor/crashpad/tools/generate_dump
)

target_compile_definitions(
    generate_dump
    PRIVATE
    -DNOMINMAX
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN
)

target_include_directories(
    generate_dump
    PRIVATE
    vendor/crashpad
    vendor/crashpad/compat/win
    vendor/mini_chromium
)

target_link_libraries(
    generate_dump
    PRIVATE
    crashpad_minidump
    crashpad_snapshot
    crashpad_tools
    getopt
)

set_source_files_properties(
    vendor/crashpad/util/misc/capture_context_win.asm
    PROPERTIES
    COMPILE_FLAGS "/safeseh"
)

set_source_files_properties(
    vendor/crashpad/util/win/safe_terminate_process.asm
    PROPERTIES
    COMPILE_FLAGS "/safeseh"
)
