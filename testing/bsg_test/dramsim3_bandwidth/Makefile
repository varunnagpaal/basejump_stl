export BASEJUMP_STL_DIR = $(abspath ../../..)
export BSG_CADENV_DIR = $(abspath ../../../../bsg_cadenv)
include $(BSG_CADENV_DIR)/cadenv.mk

INCDIR += +incdir+$(BASEJUMP_STL_DIR)/bsg_misc
INCDIR += +incdir+$(BASEJUMP_STL_DIR)/bsg_test


VCS_FLAGS = -full64 +lint=all,noSVA-UA,noSVA-NSVU,noVCDE +v2k -cpp g++
VCS_FLAGS += -sverilog -full64 -timescale=1ps/1ps +vcs+vcdpluson

SIMV = $(abspath simv)

BENCHMARKS = random_full constrained_random unit unit_load unit_load_conflict

all: run_all

run_all: $(addsuffix .run, $(BENCHMARKS))



CXXFLAGS = -std=c++11 -D_GNU_SOURCE -Wall -fPIC -shared
CXXFLAGS += -I$(BASEJUMP_STL_DIR)/imports/DRAMSim3/src
CXXFLAGS += -I$(BASEJUMP_STL_DIR)/imports/DRAMSim3/ext/headers
CXXFLAGS += -I$(BASEJUMP_STL_DIR)/imports/DRAMSim3/ext/fmt/include
CXXFLAGS += -DFMT_HEADER_ONLY=1
CXXFLAGS += -DCMD_TRACE
CXXFLAGS += -DBASEJUMP_STL_DIR=$(BASEJUMP_STL_DIR)
DRAMSIM3_SRC =  $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/bankstate.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/channel_state.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/command_queue.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/common.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/configuration.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/controller.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/dram_system.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/hmc.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/memory_system.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/refresh.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/simple_stats.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/imports/DRAMSim3/src/timing.cc
DRAMSIM3_SRC += $(BASEJUMP_STL_DIR)/bsg_test/bsg_dramsim3.cpp

libdramsim3.so:
	$(CXX) $(CXXFLAGS) -o libdramsim3.so $(DRAMSIM3_SRC)

simv:
	vcs $(VCS_FLAGS) -f sv.include -l vcs.log $(INCDIR)

out/%/trace.tr: %.py
	mkdir -p out
	mkdir -p out/$*
	python $*.py > $@

%.run: simv libdramsim3.so out/%/trace.tr
	(cd out/$*; $(SIMV) -l simv.log -sv_root $(CURDIR) -sv_lib libdramsim3)

%.dve:
	dve -full64 -vpd out/$*/vcdplus.vpd &

summary:
	@$(foreach benchmark, $(BENCHMARKS), grep -H --color -e "BSG_FINISH" -e "BSG_FATAL" \
		-e "Error" -e "BSG_ERROR" out/$(benchmark)/simv.log;)

clean:
	rm -rf csrc simv.daidir simv ucli.key vcdplus.vpd vc_hdrs.h DVEfiles
	rm -f vcs.log bsg_nonsynth_dramsim3_trace.txt dramsim3epoch.json
	rm -f trace.tr *.trace *.pyc
	rm -rf out