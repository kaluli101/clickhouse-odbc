find_program(PERL perl)
find_program(PYTHON python2 python python3)
find_program(SQLCMD sqlcmd)

set(PERL_USES_MDAC 0)
set(PERL_USES_UNIXODBC 0)
set(PERL_USES_IODBC 0)

if(PERL)
    execute_process(
        COMMAND ${PERL} ${CMAKE_CURRENT_SOURCE_DIR}/detect_driver_manager.pl
        OUTPUT_VARIABLE _perl_output
        OUTPUT_STRIP_TRAILING_WHITESPACE
#       ERROR_QUIET
    )
    message(STATUS "ODBC Driver Manager used by Perl's DBD::ODBC: ${_perl_output}")
    if(_perl_output MATCHES "Microsoft")
        set(PERL_USES_MDAC 1)
    endif()
    if(_perl_output MATCHES "unixODBC")
        set(PERL_USES_UNIXODBC 1)
    endif()
    if(_perl_output MATCHES "iODBC")
        set(PERL_USES_IODBC 1)
    endif()
endif()

set(PYTHON_USES_MDAC 0)
set(PYTHON_USES_UNIXODBC 0)
set(PYTHON_USES_IODBC 0)

if(PYTHON)
    execute_process(
        COMMAND ${PYTHON} ${CMAKE_CURRENT_SOURCE_DIR}/detect_driver_manager.py
        OUTPUT_VARIABLE _python_output
        OUTPUT_STRIP_TRAILING_WHITESPACE
#       ERROR_QUIET
    )
    message(STATUS "ODBC Driver Manager used by Python's pyodbc: ${_python_output}")
    if(_python_output MATCHES "Microsoft")
        set(PYTHON_USES_MDAC 1)
    endif()
    if(_python_output MATCHES "unixODBC")
        set(PYTHON_USES_UNIXODBC 1)
    endif()
    if(_python_output MATCHES "iODBC")
        set(PYTHON_USES_IODBC 1)
    endif()
endif()

message(STATUS "Testing with:\n\t"
    "TEST_DSN_LIST=${TEST_DSN_LIST}\n\t"
    "PERL=${PERL}\n\t"
    "PERL_USES_MDAC=${PERL_USES_MDAC}\n\t"
    "PERL_USES_UNIXODBC=${PERL_USES_UNIXODBC}\n\t"
    "PERL_USES_IODBC=${PERL_USES_IODBC}\n\t"
    "PYTHON=${PYTHON}\n\t"
    "PYTHON_USES_MDAC=${PYTHON_USES_MDAC}\n\t"
    "PYTHON_USES_UNIXODBC=${PYTHON_USES_UNIXODBC}\n\t"
    "PYTHON_USES_IODBC=${PYTHON_USES_IODBC}\n\t"
    "ODBC_UNIXODBC_ISQL=${ODBC_UNIXODBC_ISQL}\n\t"
    "ODBC_UNIXODBC_IUSQL=${ODBC_UNIXODBC_IUSQL}\n\t"
    "ODBC_IODBC_IODBCTEST=${ODBC_IODBC_IODBCTEST}\n\t"
    "ODBC_IODBC_IODBCTESTW=${ODBC_IODBC_IODBCTESTW}\n\t"
    "SQLCMD=${SQLCMD}"
)

set(SQL_QUERY "123456+456789" CACHE STRING "")
set(SQL_ANSWER "580245" CACHE STRING "")

set(test_dsn_counter 0)
foreach (test_dsn ${TEST_DSN_LIST})
    if (test_dsn)
        if(PERL AND (
                ("${ODBC_PROVIDER}" STREQUAL "MDAC" AND PERL_USES_MDAC) OR
                ("${ODBC_PROVIDER}" STREQUAL "UnixODBC" AND PERL_USES_UNIXODBC) OR
                ("${ODBC_PROVIDER}" STREQUAL "iODBC" AND PERL_USES_IODBC)
        ))
            add_test(NAME "test.pl-a-dsn-${test_dsn_counter}" COMMAND ${PERL} ${CMAKE_CURRENT_SOURCE_DIR}/test.pl "${test_dsn}" a)
            add_test(NAME "test.pl-w-dsn-${test_dsn_counter}" COMMAND ${PERL} ${CMAKE_CURRENT_SOURCE_DIR}/test.pl "${test_dsn}" w)
        endif()
        if(PYTHON AND (
                ("${ODBC_PROVIDER}" STREQUAL "MDAC" AND PYTHON_USES_MDAC) OR
                ("${ODBC_PROVIDER}" STREQUAL "UnixODBC" AND PYTHON_USES_UNIXODBC) OR
                ("${ODBC_PROVIDER}" STREQUAL "iODBC" AND PYTHON_USES_IODBC)
        ))
            add_test(NAME "test.py-a-dsn-${test_dsn_counter}" COMMAND ${PYTHON} ${CMAKE_CURRENT_SOURCE_DIR}/test.py "${test_dsn}" a)
            add_test(NAME "test.py-w-dsn-${test_dsn_counter}" COMMAND ${PYTHON} ${CMAKE_CURRENT_SOURCE_DIR}/test.py "${test_dsn}" w)
        endif()
        if(ODBC_UNIXODBC_ISQL AND "${ODBC_PROVIDER}" STREQUAL "UnixODBC")
            add_test(NAME "isql-dsn-${test_dsn_counter}" COMMAND sh -c "echo select ${SQL_QUERY} | ${ODBC_UNIXODBC_ISQL} '${test_dsn}' | grep ${SQL_ANSWER}")
            add_test(NAME "test.sh-isql-dsn-${test_dsn_counter}" COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/test.sh ${ODBC_UNIXODBC_ISQL} \"${test_dsn}\" -v -b)
        endif()
        if(ODBC_UNIXODBC_IUSQL AND "${ODBC_PROVIDER}" STREQUAL "UnixODBC")
            add_test(NAME "iusql-dsn-${test_dsn_counter}" COMMAND sh -c "echo select ${SQL_QUERY} | ${ODBC_UNIXODBC_IUSQL} '${test_dsn}' | grep ${SQL_ANSWER}")
            add_test(NAME "test.sh-iusql-dsn-${test_dsn_counter}" COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/test.sh ${ODBC_UNIXODBC_IUSQL} \"${test_dsn}\" -v -b)
        endif()
        if(ODBC_IODBC_IODBCTEST AND "${ODBC_PROVIDER}" STREQUAL "iODBC")
            add_test(NAME "iodbctest-dsn-${test_dsn_counter}" COMMAND sh -c "echo select ${SQL_QUERY} | ${ODBC_IODBC_IODBCTEST} 'DSN=${test_dsn}' | grep ${SQL_ANSWER}")
            add_test(NAME "test.sh-iodbctest-dsn-${test_dsn_counter}" COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/test.sh ${ODBC_IODBC_IODBCTEST} \"DSN=${test_dsn}\")
        endif()
        if(ODBC_IODBC_IODBCTESTW AND "${ODBC_PROVIDER}" STREQUAL "iODBC")
            add_test(NAME "iodbctestw-dsn-${test_dsn_counter}" COMMAND sh -c "echo select ${SQL_QUERY} | ${ODBC_IODBC_IODBCTESTW} 'DSN=${test_dsn}' | grep ${SQL_ANSWER}")
            add_test(NAME "test.sh-iodbctestw-dsn-${test_dsn_counter}" COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/test.sh ${ODBC_IODBC_IODBCTESTW} \"DSN=${test_dsn}\")
        endif()

        math (EXPR test_dsn_counter "${test_dsn_counter}+1")
    endif ()
endforeach ()

if(SQLCMD)
    # MS SQL server need change server in file:
    add_test(NAME "sqlcmd" COMMAND ${SQLCMD} -i ${CMAKE_CURRENT_SOURCE_DIR}/mssql.linked.server.sql)
endif()
