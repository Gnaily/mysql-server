# Copyright (c) 2015, 2020, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2.0,
# as published by the Free Software Foundation.
#
# This program is also distributed with certain software (including
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA

INCLUDE_DIRECTORIES(SYSTEM ${GMOCK_INCLUDE_DIRS})

IF(NOT BUILD_IS_SINGLE_CONFIG)
  FOREACH(config ${CMAKE_CONFIGURATION_TYPES})
    EXECUTE_PROCESS(
      COMMAND ${CMAKE_COMMAND} -E
      make_directory ${CMAKE_CURRENT_BINARY_DIR}/${config}/var
      COMMAND ${CMAKE_COMMAND} -E
      make_directory ${CMAKE_CURRENT_BINARY_DIR}/${config}/var/log
      COMMAND ${CMAKE_COMMAND} -E
      make_directory ${CMAKE_CURRENT_BINARY_DIR}/${config}/var/log/harness
    )
  ENDFOREACH()
ELSE()
  EXECUTE_PROCESS(
    COMMAND ${CMAKE_COMMAND} -E
    make_directory ${CMAKE_CURRENT_BINARY_DIR}/var
    COMMAND ${CMAKE_COMMAND} -E
    make_directory ${CMAKE_CURRENT_BINARY_DIR}/var/log
    COMMAND ${CMAKE_COMMAND} -E
    make_directory ${CMAKE_CURRENT_BINARY_DIR}/var/log/harness
  )
ENDIF()

ADD_SUBDIRECTORY(plugins)

SET(TEST_MODULE harness)

SET(TESTS
  test_utility_string.cc
  test_loader.cc
  test_loader_lifecycle.cc
  test_logging.cc
  test_designator.cc
  test_iterator.cc
  test_utilities.cc
  test_common.cc
  test_config.cc
  test_filesystem.cc
  test_hostname_validator.cc
  test_ip_address.cc
  test_bug22104451.cc
  test_resolver.cc
  test_random_generator.cc
  test_mysql_router_thread.cc
  test_mpmc_queue.cc
  test_dynamic_state.cc
  test_arg_handler.cc
  test_vt100.cc
  test_string_utils.cc
  test_process_launcher.cc
)

FOREACH(TEST ${TESTS})
  ADD_TEST_FILE(${TEST} MODULE ${TEST_MODULE}
    INCLUDE_DIRS ${MySQLRouter_SOURCE_DIR}/src/harness/shared/include/
    INCLUDE_DIRS ${MySQLRouter_SOURCE_DIR}/src/harness/src/
    INCLUDE_DIRS ${MySQLRouter_SOURCE_DIR}/src/harness/tests/include/
    INCLUDE_DIRS ${MySQLRouter_SOURCE_DIR}/tests/
    LIB_DEPENDS test-helpers;harness_stdx_filesystem)
ENDFOREACH()

SET(TESTS
  test_keyring.cc
  test_keyring_manager.cc
)

FOREACH(TEST ${TESTS})
  ADD_TEST_FILE(${TEST} MODULE ${TEST_MODULE}
    INCLUDE_DIRS ${MySQLRouter_SOURCE_DIR}/src/harness/shared/include/
    LIB_DEPENDS test-helpers;${SSL_LIBRARIES})
ENDFOREACH()

SET(TESTS
  test_stdx_bit.cc
  test_stdx_expected.cc
  test_stdx_string_view.cc
  test_net_ts_buffer.cc
  test_net_ts_impl_file.cc
  test_net_ts_impl_resolver.cc
  test_net_ts_impl_socket.cc
)

FOREACH(TEST ${TESTS})
  ADD_TEST_FILE(${TEST} MODULE ${TEST_MODULE}
    LIB_DEPENDS harness_net_ts)
ENDFOREACH()

SET(TESTS
  test_stdx_io_file_handle.cc
)

FOREACH(TEST ${TESTS})
  ADD_TEST_FILE(${TEST} MODULE ${TEST_MODULE}
    LIB_DEPENDS harness_stdx_io)
ENDFOREACH()


# Use configuration file templates to generate configuration files
CONFIGURE_TEST_FILE_TEMPLATES(${CMAKE_CURRENT_SOURCE_DIR}/data
  "tests-good-1.cfg.in;tests-good-2.cfg.in;tests-start-1.cfg.in")
SET(TEST_FILES
  logger.cfg
  logger.d/default.cfg
  logger.d/magic.cfg
  logger.d/one.cfg
  magic-alt.cfg
  tests-bad-1.cfg
  tests-bad-2.cfg
  tests-bad-3.cfg
  )
COPY_TEST_FILES(${CMAKE_CURRENT_SOURCE_DIR}/data "${TEST_FILES}")
