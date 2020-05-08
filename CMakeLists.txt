cmake_minimum_required(VERSION 3.16)

project(RBUDP)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(QUANTANET_SOURCES
	QUANTAnet_rbudpBase_c.cxx
	QUANTAnet_rbudpBase_c.hxx
	QUANTAnet_rbudpReceiver_c.cxx
	QUANTAnet_rbudpReceiver_c.hxx
	QUANTAnet_rbudpSender_c.cxx
	QUANTAnet_rbudpSender_c.hxx	
)

add_executable(recvfile recvfile.cxx ${QUANTANET_SOURCES})
add_executable(sendfile sendfile.cxx ${QUANTANET_SOURCES})
add_executable(urecv urecv.cxx ${QUANTANET_SOURCES})
add_executable(usend usend.cxx ${QUANTANET_SOURCES})
add_executable(recvfilelist recvfilelist.cxx ${QUANTANET_SOURCES})
add_executable(sendfilelist sendfilelist.cxx ${QUANTANET_SOURCES})
add_executable(recvstream recvstream.cxx ${QUANTANET_SOURCES})
add_executable(sendstream sendstream.cxx ${QUANTANET_SOURCES})