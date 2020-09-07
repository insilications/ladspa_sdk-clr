###############################################################################
#
# Installation DIRECTORIES
#
# Change these if you want to install somewhere else.

INSTALL_PLUGINS_DIR	=	/usr/lib64/ladspa/
INSTALL_INCLUDE_DIR	=	/usr/include/
INSTALL_BINARY_DIR	=	/usr/bin/

###############################################################################
#
# GENERAL
#

# CFLAGS	=	-Wall -Werror -O3
# CXXFLAGS	=	-Wall -Werror -O3

INCLUDES	=	-I. -DDEFAULT_LADSPA_PATH=$(INSTALL_PLUGINS_DIR)
DYNAMIC_LD_LIBS =	-ldl
LIBRARIES	=	$(DYNAMIC_LD_LIBS) -lm
PLUGINS		=	../plugins/amp.so				\
			../plugins/delay.so				\
			../plugins/filter.so				\
			../plugins/noise.so				\
			../plugins/sine.so				\
			../plugins/amp.a				\
			../plugins/delay.a				\
			../plugins/filter.a				\
			../plugins/noise.a				\
			../plugins/sine.a				
PROGRAMS	=	../bin/analyseplugin				\
			../bin/applyplugin 				\
			../bin/listplugins
CC		=	cc
CXX		=	c++

###############################################################################
#
# RULES TO BUILD PLUGINS FROM C OR C++ CODE
#

../plugins/filter.so:	plugins/filter.c ladspa.h gcc_exports.map
	$(CC) $(CFLAGS) $(INCLUDES) -fPIC -o plugins/filter.o -c plugins/filter.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o ../plugins/filter.so plugins/filter.o -nostartfiles -shared -lm

../plugins/%.so:	plugins/%.c ladspa.h gcc_exports.map
	$(CC) $(CFLAGS) $(INCLUDES) -fPIC -o plugins/$*.o -c plugins/$*.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o ../plugins/$*.so plugins/$*.o -nostartfiles -shared

../plugins/%.so:	plugins/%.cpp ladspa.h gcc_exports.map
	$(CXX) $(CXXFLAGS) $(INCLUDES) -fPIC -o plugins/$*.o -c plugins/$*.cpp
	$(CXX) $(LDFLAGS) -o ../plugins/$*.so plugins/$*.o -shared

../plugins/filter.a: plugins/filter.c ladspa.h gcc_exports.map
	$(CC) $(CFLAGS) $(INCLUDES) -fPIC -o plugins/filter.o -c plugins/filter.c
	$(AR) rcs ../plugins/filter.a plugins/filter.o

../plugins/%.a: plugins/%.c ladspa.h gcc_exports.map
	$(CC) $(CFLAGS) $(INCLUDES) -fPIC -o plugins/$*.o -c plugins/$*.c
	$(AR) rcs ../plugins/$*.a plugins/$*.o

../plugins/%.a: plugins/%.cpp ladspa.h gcc_exports.map
	$(CXX) $(CXXFLAGS) $(INCLUDES) -fPIC -o plugins/$*.o -c plugins/$*.cpp
	$(AR) rcs ../plugins/$*.a plugins/$*.o


###############################################################################
#
# TARGETS
#

test:	/tmp/test.wav ../snd/noise.wav always
	@echo ---------------------------------------------
	@echo First listen to the white noise input signal:
	@echo ---------------------------------------------
	@echo -sndfile-play ../snd/noise.wav
	@echo -------------------------
	@echo Compare to plugin output.
	@echo -------------------------
	@echo Should be a noise band around 6000Hz, repeated quietly after 1s.
	-sndfile-play /tmp/test.wav
	@echo Test complete.

install:	targets
	-mkdir -p $(INSTALL_PLUGINS_DIR)
	-mkdir -p $(INSTALL_INCLUDE_DIR)
	-mkdir -p $(INSTALL_BINARY_DIR)
	cp ../plugins/* $(INSTALL_PLUGINS_DIR)
	cp ladspa.h $(INSTALL_INCLUDE_DIR)
	cp ../bin/* $(INSTALL_BINARY_DIR)

/tmp/test.wav:	targets ../snd/noise.wav
	../bin/listplugins
	../bin/analyseplugin ../plugins/filter.so
	../bin/analyseplugin ../plugins/delay.so
	../bin/analyseplugin ../plugins/sine.so
	echo ; ../bin/analyseplugin -l ../plugins/sine.so ; echo
	../bin/analyseplugin ../plugins/amp.so
	../bin/analyseplugin ../plugins/noise.so
	../bin/applyplugin	-s 1					\
				../snd/noise.wav /tmp/test.wav		\
				../plugins/filter.so lpf 500		\
				../plugins/filter.so lpf 500		\
				../plugins/sine.so sine_fcaa 6000	\
				../plugins/delay.so delay_5s 1 0.1	\
				../plugins/amp.so amp_mono 4		\

targets:	$(PLUGINS) $(PROGRAMS)

###############################################################################
#
# PROGRAMS
#

../bin/applyplugin:	applyplugin.o load.o default.o
	$(CC) $(CFLAGS) $(INCLUDES) $(LDFLAGS)				\
		-o ../bin/applyplugin					\
		applyplugin.o load.o default.o				\
		$(LIBRARIES)

../bin/analyseplugin:	analyseplugin.o load.o default.o
	$(CC) $(CFLAGS) $(INCLUDES) $(LDFLAGS)				\
		-o ../bin/analyseplugin 				\
		analyseplugin.o load.o default.o			\
		$(LIBRARIES)

../bin/listplugins:	listplugins.o search.o
	$(CC) $(CFLAGS) $(INCLUDES) $(LDFLAGS)				\
		-o ../bin/listplugins	 				\
		listplugins.o search.o					\
		$(LIBRARIES)

###############################################################################
#
# UTILITIES
#

always:	

clean:
	-rm -f `find . -name "*.o"` ../bin/* ../plugins/*
	-rm -f `find .. -name "*~"`
	-rm -f *.bak core score.srt
	-rm -f *.bb *.bbg *.da *-ann gmon.out bb.out
	-rm -f `find .. -name "*.class"`

backup:		clean
	(cd ../../;							\
	tar czf `date '+../backup/ladspa_sdk.%Y%m%d%H%M.tgz'` ladspa_sdk/)

###############################################################################

