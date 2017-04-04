#
#   Copyright (c) 2012-2013 Martin Sustrik  All rights reserved.
#   Copyright (c) 2013 GoPivotal, Inc.  All rights reserved.
#   Copyright (c) 2015-2016 Jack R. Dunaway. All rights reserved.
#   Copyright 2016 Garrett D'Amore <garrett@damore.org>
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom
#   the Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
#   IN THE SOFTWARE.
#

set (NN_SOURCES
    nn.h
    tcp.h
    pipeline.h

    core/ep.h
    core/ep.c
    core/global.h
    core/global.c
    core/pipe.c
    core/poll.c
    core/sock.h
    core/sock.c
    core/sockbase.c
    core/symbol.c

    aio/ctx.h
    aio/ctx.c
    aio/fsm.h
    aio/fsm.c
    aio/pool.h
    aio/pool.c
    aio/timer.h
    aio/timer.c
    aio/timerset.h
    aio/timerset.c
    aio/usock.h
    aio/usock.c
    aio/worker.h
    aio/worker.c

    utils/alloc.h
    utils/alloc.c
    utils/atomic.h
    utils/atomic.c
    utils/attr.h
    utils/chunk.h
    utils/chunk.c
    utils/chunkref.h
    utils/chunkref.c
    utils/clock.h
    utils/clock.c
    utils/closefd.h
    utils/closefd.c
    utils/cont.h
    utils/efd.h
    utils/efd.c
    utils/err.h
    utils/err.c
    utils/fast.h
    utils/fd.h
    utils/hash.h
    utils/hash.c
    utils/list.h
    utils/list.c
    utils/msg.h
    utils/msg.c
    utils/condvar.h
    utils/condvar.c
    utils/mutex.h
    utils/mutex.c
    utils/once.h
    utils/once.c
    utils/queue.h
    utils/queue.c
    utils/random.h
    utils/random.c
    utils/sem.h
    utils/sem.c
    utils/sleep.h
    utils/sleep.c
    utils/strcasecmp.c
    utils/strcasecmp.h
    utils/strcasestr.c
    utils/strcasestr.h
    utils/strncasecmp.c
    utils/strncasecmp.h
    utils/thread.h
    utils/thread.c
    utils/wire.h
    utils/wire.c

    devices/device.h
    devices/device.c

    protocols/utils/dist.h
    protocols/utils/dist.c
    protocols/utils/excl.h
    protocols/utils/excl.c
    protocols/utils/fq.h
    protocols/utils/fq.c
    protocols/utils/lb.h
    protocols/utils/lb.c
    protocols/utils/priolist.h
    protocols/utils/priolist.c


    protocols/pipeline/push.c
    protocols/pipeline/pull.c
    protocols/pipeline/xpull.h
    protocols/pipeline/xpull.c
    protocols/pipeline/xpush.h
    protocols/pipeline/xpush.c


    transports/utils/backoff.h
    transports/utils/backoff.c
    transports/utils/dns.h
    transports/utils/dns.c
    transports/utils/dns_getaddrinfo.h
    transports/utils/dns_getaddrinfo.inc
    transports/utils/dns_getaddrinfo_a.h
    transports/utils/dns_getaddrinfo_a.inc
    transports/utils/iface.h
    transports/utils/iface.c
    transports/utils/literal.h
    transports/utils/literal.c
    transports/utils/port.h
    transports/utils/port.c
    transports/utils/streamhdr.h
    transports/utils/streamhdr.c
    transports/utils/base64.h
    transports/utils/base64.c

 

    transports/tcp/atcp.h
    transports/tcp/atcp.c
    transports/tcp/btcp.h
    transports/tcp/btcp.c
    transports/tcp/ctcp.h
    transports/tcp/ctcp.c
    transports/tcp/stcp.h
    transports/tcp/stcp.c
    transports/tcp/tcp.c


)

if (WIN32)
    list (APPEND NN_SOURCES
        aio/usock_win.h
        aio/usock_win.inc
        aio/worker_win.h
        aio/worker_win.inc
        utils/thread_win.h
        utils/thread_win.inc
        utils/win.h
    )
elseif (UNIX)
    list (APPEND NN_SOURCES
        aio/usock_posix.h
        aio/usock_posix.inc
        aio/worker_posix.h
        aio/worker_posix.inc
        utils/thread_posix.h
        utils/thread_posix.inc
    )
else ()
    message (FATAL_ERROR "Assertion failed; this path is unreachable.")
endif ()

if (NN_HAVE_EPOLL)
    add_definitions (-DNN_USE_EPOLL)
    list (APPEND NN_SOURCES
        aio/poller.h
        aio/poller.c
        aio/poller_epoll.h
        aio/poller_epoll.inc
    )
elseif (NN_HAVE_KQUEUE)
    add_definitions (-DNN_USE_KQUEUE)
    list (APPEND NN_SOURCES
        aio/poller.h
        aio/poller.c
        aio/poller_kqueue.h
        aio/poller_kqueue.inc
    )
elseif (NN_HAVE_POLL)
    add_definitions (-DNN_USE_POLL)
    list (APPEND NN_SOURCES
        aio/poller.h
        aio/poller.c
        aio/poller_poll.h
        aio/poller_poll.inc
    )
elseif (NN_HAVE_WINSOCK)
    # No operation
else ()
    message (SEND_ERROR "ERROR: could not determine socket polling method.")
    message (SEND_ERROR "${ISSUE_REPORT_MSG}" )
endif ()

if (NN_HAVE_EVENTFD)
    add_definitions (-DNN_USE_EVENTFD)
    list (APPEND NN_SOURCES
        utils/efd_eventfd.h
        utils/efd_eventfd.inc
    )
elseif (NN_HAVE_PIPE)
    add_definitions (-DNN_USE_PIPE)
    list (APPEND NN_SOURCES
        utils/efd_pipe.h
        utils/efd_pipe.inc
    )
elseif (NN_HAVE_SOCKETPAIR)
    add_definitions (-DNN_USE_SOCKETPAIR)
    list (APPEND NN_SOURCES
        utils/efd_socketpair.h
        utils/efd_socketpair.inc
    )
elseif (NN_HAVE_WINSOCK)
    add_definitions (-DNN_USE_WINSOCK)
    list (APPEND NN_SOURCES
        utils/efd_win.h
        utils/efd_win.inc
    )
else ()
    message (SEND_ERROR "ERROR: could not determine socket signaling method.")
    message (SEND_ERROR "${ISSUE_REPORT_MSG}" )
endif ()

# Provide same folder structure in IDE as on disk
foreach (f ${NN_SOURCES})
    # Get the path of the file relative to source directory
    if (IS_ABSOLUTE "${f}")
        file (RELATIVE_PATH f ${CMAKE_CURRENT_SOURCE_DIR} ${f})
    endif ()
    set (SRC_GROUP "${f}")
    set (f "${CMAKE_CURRENT_SOURCE_DIR}/${f}")

    # Remove the filename part
    string (REGEX REPLACE "(.*)(/[^/]*)$" "\\1" SRC_GROUP ${SRC_GROUP})

    # CMake source_group expects \\, not /
    string (REPLACE / \\ SRC_GROUP ${SRC_GROUP})
    source_group ("${SRC_GROUP}" FILES ${f})
endforeach ()

if (NN_STATIC_LIB)
    add_library (${PROJECT_NAME} STATIC ${NN_SOURCES})
else ()
    add_library (${PROJECT_NAME} SHARED ${NN_SOURCES})
    add_definitions (-DNN_SHARED_LIB)
    set_target_properties (${PROJECT_NAME} PROPERTIES
        SOVERSION "${NN_ABI_VERSION}")
endif ()

# Set library outputs same as top-level project binary outputs
set_target_properties (${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set_target_properties (${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set_target_properties (${PROJECT_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

target_link_libraries (${PROJECT_NAME} ${NN_REQUIRED_LIBRARIES})
if(THREADS_HAVE_PTHREAD_ARG)
    add_definitions (-pthread)
endif()
if(CMAKE_THREAD_LIBS_INIT)
    target_link_libraries (${PROJECT_NAME} "${CMAKE_THREAD_LIBS_INIT}")
endif()

# pkg-config file
if(NN_REQUIRED_LIBRARIES)
    foreach (lib ${NN_REQUIRED_LIBRARIES})
        set (NN_REQUIRED_LFLAGS "${NN_REQUIRED_LFLAGS} -l${lib}")
    endforeach()
endif()
configure_file (pkgconfig.in ${PROJECT_NAME}.pc @ONLY)
install (
    FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
install (TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
