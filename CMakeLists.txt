set(PROTO_DIR ${PROJECT_DIR}/proto_src)
set(GEN_C_DIR ${COMPONENT_DIR}/generated)

# Check if protobuf is installed in the system
find_program(PROTOC_EXECUTABLE protoc)
if(NOT PROTOC_EXECUTABLE)
	message(FATAL_ERROR "protoc not found. Please install protoc (Protocol Buffers compiler).")
endif()

# Check if protobuf-c plugin is installed in the system
find_program(PROTOC_C_PLUGIN protoc-c)
if(NOT PROTOC_C_PLUGIN)
	message(FATAL_ERROR "protoc-c not found. Please install protobuf-c-compiler.")
endif()

# Collect all proto files
file(GLOB PROTO_FILES "${PROTO_DIR}/*.proto")

if(NOT PROTO_FILES)
	message(FATAL_ERROR "No .proto files found in ${PROTO_DIR}")
endif()

file(MAKE_DIRECTORY ${GEN_C_DIR})

# Generate protobuf-c files from .proto definitions
execute_process(
	COMMAND protoc --proto_path=${PROTO_DIR} --c_out=${GEN_C_DIR} ${PROTO_FILES}
	RESULT_VARIABLE result
)
if(NOT result EQUAL 0)
	message(FATAL_ERROR "protoc-c failed for vaccel protos")
endif()

# Get generated sources
file(GLOB GEN_C_SOURCES "${GEN_C_DIR}/*.pb-c.c")

# Register the generated files and the protobuf-c component with our custom vaccel_rpc_proto component
idf_component_register(
	SRCS
		${GEN_C_SOURCES}
	INCLUDE_DIRS
		${GEN_C_DIR}
	REQUIRES
		protobuf-c
)
