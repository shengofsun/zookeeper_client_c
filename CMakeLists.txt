cmake_minimum_required(VERSION 2.8)

PROJECT(dsn.zookeeper)

set(src 
"src/hashtable/hashtable_itr.c"
"src/hashtable/hashtable.c"
"src/mt_adaptor.c"
"src/recordio.c"
"src/zk_hashtable.c"
"src/zk_log.c"
"src/zookeeper.c"
)

set(src_generated "generated/zookeeper.jute.c")

ADD_DEFINITIONS(-DTHREADED)
if (NOT WIN32)
	ADD_DEFINITIONS(-fPIC)
endif()

INCLUDE_DIRECTORIES("include")
ADD_LIBRARY(zookeeper_mt STATIC ${src} ${src_generated})
