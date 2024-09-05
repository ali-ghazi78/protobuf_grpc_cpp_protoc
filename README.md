# protobuf_grpc_cpp_protoc
This Docker image has glog, protoc, protobuf and cmake for building CPP application.

https://hub.docker.com/repository/docker/alighazi1/protobuf_grpc_cpp_protoc/general


Below is a sample Makefile from https://github.com/protocolbuffers/protobuf/  which has been modified for our image. 


```
PROTOBUF_ABSL_DEPS = absl_absl_check absl_absl_log absl_algorithm absl_base absl_bind_front absl_bits absl_btree absl_cleanup absl_cord absl_core_headers absl_debugging absl_die_if_null absl_dynamic_annotations absl_flags absl_flat_hash_map absl_flat_hash_set absl_function_ref absl_hash absl_layout absl_log_initialize absl_log_severity absl_memory absl_node_hash_map absl_node_hash_set absl_optional absl_span absl_status absl_statusor absl_strings absl_synchronization absl_time absl_type_traits absl_utility absl_variant

PROTOBUF_UTF8_RANGE_LINK_LIBS = -lutf8_validity
LIBS += -L/usr/local/include/grpc++ -lgrpc++
export PKG_CONFIG_PATH = /.local/lib/pkgconfig:./grpc/third_party/re2:/.local/bin/:/.local/share/pkgconfig/

HOST_SYSTEM = $(shell uname | cut -f 1 -d_)
SYSTEM ?= $(HOST_SYSTEM)
CXX = g++
CPPFLAGS += `pkg-config --cflags protobuf grpc absl_flags absl_flags_parse`
CXXFLAGS += -std=c++17

ifeq ($(SYSTEM),Darwin)
LDFLAGS += -L/usr/local/lib `pkg-config --libs --static protobuf grpc++ absl_flags absl_flags_parse $(PROTOBUF_ABSL_DEPS)`\
           $(PROTOBUF_UTF8_RANGE_LINK_LIBS) \
           -pthread\
           -lgrpc++_reflection\
           -ldl 
else
LDFLAGS += -L/usr/local/lib `pkg-config --libs --static protobuf grpc++ absl_flags absl_flags_parse $(PROTOBUF_ABSL_DEPS)`\
           $(PROTOBUF_UTF8_RANGE_LINK_LIBS) \
           -pthread\
           -Wl,--no-as-needed -lgrpc++_reflection -Wl,--as-needed\
           -ldl -lglog 
endif
PROTOC = protoc
GRPC_CPP_PLUGIN = grpc_cpp_plugin
# GRPC_CPP_PLUGIN_PATH ?= `which $(GRPC_CPP_PLUGIN)`
GRPC_CPP_PLUGIN_PATH = /.local/bin/grpc_cpp_plugin

#PROTOS_PATH = ../../protos
PROTOS_PATH = .

vpath %.proto $(PROTOS_PATH)

all:  tsd tsc


tsc: client.o sns.pb.o sns.grpc.pb.o tsc.o
	$(CXX) $^ $(LDFLAGS) -g -o $@

tsd: sns.pb.o sns.grpc.pb.o tsd.o
	$(CXX) $^ $(LDFLAGS) -g -o $@


.PRECIOUS: %.grpc.pb.cc
%.grpc.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --grpc_out=. --plugin=protoc-gen-grpc=$(GRPC_CPP_PLUGIN_PATH) $<

.PRECIOUS: %.pb.cc
%.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --cpp_out=. $<

clean:
	rm -f *~ *.o *.pb.cc *.pb.h tsc tsd

```


the most important thing is this path that has been exported according to our docker imge 
```
export PKG_CONFIG_PATH = /.local/lib/pkgconfig:./grpc/third_party/re2:/.local/bin/:/.local/share/pkgconfig/

```
you can change the files to compile accordingly. 

```
docker run -v .:/home  -w /home   -i -t alighazi1/protobuf_grpc_cpp_protoc:latest /bin/bash
```
then you will enter the shell of the container and you can use the make or make clean commands to make the files.

