
ifeq ($(LIBHOME),)
LIBHOME:=$(FEPHOME)/lib
endif

TESTF := $(word 3,$(shell bash -c "grep FEPLIB_VERSION $(LIBHOME)/sysident.h"))
TEST510 = $(shell bash -c "if [ "$(TESTF)" -gt 510 ]; then echo yes; fi")
TEST700 = $(shell bash -c "if [ "$(TESTF)" -ge 700 ]; then echo yes; fi")
TEST750 = $(shell bash -c "if [ "$(TESTF)" -ge 750 ]; then echo yes; fi")

DBGTEST=$(wildcard debug_build)
ROOTTEST= $(strip $(shell bash -c "ls C:/ 2>/dev/null"))
ifeq ($(ROOTTEST),)
  OSTYPE:=unix
  OSPLAT:=$(shell bash -c "uname -m")
  ifneq ($(findstring arm,$(OSPLAT)),)
    OSPLAT:=arm
  else
    ifeq ($(shell bash -c "uname -s"),SunOS)
      OSPLAT:=sun
    else
      OSPLAT:=linux86
    endif
  endif
else
  OSTYPE:=win
  OSPLAT:=win
endif

ifeq ($(OSTYPE),unix)
  VERSION_OBJ:=
  LIBHOME_CYG:=$(LIBHOME)
  CCFIN:=-c
  CCFOUT:=-o
  CC=gcc
  COMFLAGS=-I$(FEPHOME)/lib -I. -O2
  ifeq ($(OSPLAT),sun)
    LLIBS=-lsocket -lnsl -ldl -lm -lelf
    CFLAGS=-DSYS_SUNOS=8 $(COMFLAGS)
    LDFLAGS=-mimpure-text
  else
    ifeq ($(OSPLAT),linux86)
      LLIBS=-lm -lpam -ldl -lpthread
      CFLAGS=-DSYS_LINUX $(COMFLAGS)
      LDFLAGS=
    else
      ifeq ($(OSPLAT),arm)
        LLIBS=-lm -lpam -ldl -lpthread
        CFLAGS=-DSYS_ARM -fPIC $(COMFLAGS)
        LDFLAGS=
      endif
    endif
  endif
  LINK=gcc $(LDFLAGS) -s -o 
  LINKSO=gcc $(LDFLAGS) -s -shared -o 
  MT=echo 

  OBJEXT=.o
  LIBEXT=.a
  SOEXT=.so
  EXEEXT=
  VCDLLS=
else
  VERSION_OBJ:=version.obj
  LIBHOME_CYG :=$(shell bash -c "cygpath -au $(LIBHOME)")
  CCFIN:=/TC -c 
  CCFOUT:=/Fo
  CLPATHa = $(patsubst %/bin/cl.exe,%, $(shell bash -c "type -p cl.exe | tr A-Z a-z"))
  CLPATH  = $(shell bash -c "cygpath -ds \"$(CLPATHa)\"")
  CLPATHW = $(shell bash -c "cygpath -m \"$(CLPATHa)\"")
  INCS=/I "$(CLPATHW)\include" /I pthread /I unixem_include
  LBS=/LIBPATH:"$(CLPATHW)\lib"

  SOFLAGS = /dll
  CC=cl
  ifneq ($(TEST700),)
    MT=mt 
  else
    MT=echo 
  endif

  ifeq ($(DBGTEST),)
    LINK="$(CLPATHa)/bin/link" /nologo /machine:ix86 /release $(LBS) /out:
    LINKSO="$(CLPATHa)/bin/link" /nologo $(SOFLAGS) /machine:ix86 $(LBS) /release /out:
    CFLAGS=/nologo /MD /W3 /Ox /Z7 /D "NDEBUG" /D "WIN32" /D "SYS_WINNT" /D FD_SETSIZE=512 /I $(LIBHOME) /I . $(INCS) /D "_CRT_SECURE_NO_WARNINGS" /D "_CRT_SECURE_NO_DEPRECATE"
  else
# Debug enabled versions

    LINK="$(CLPATHa)/bin/link" /nologo /machine:ix86 /debug $(LBS) /fixed:no /incremental:no /out:
    LINKSO="$(CLPATHa)/bin/link" /nologo $(SOFLAGS) /machine:ix86 $(LBS) /debug /fixed:no /incremental:no /out:
    CFLAGS=/nologo /MD /W3 /Od /Z7 /D "WIN32" /D "SYS_WINNT" /D FD_SETSIZE=512 /I $(LIBHOME) /I . $(INCS) /D "_CRT_SECURE_NO_WARNINGS" /D "_CRT_SECURE_NO_DEPRECATE"
  endif


  LLIBS=msvcrt.lib oldnames.lib kernel32.lib user32.lib \
        ws2_32.lib advapi32.lib netapi32.lib winspool.lib
  OBJEXT=.obj
  LIBEXT=.lib
  SOEXT=.dll
  EXEEXT=.exe
#  VCDLLS=$(CLPATH)\bin\mspdb60.dll
endif

MTSLIB=$(wildcard $(LIBHOME)/mtsclient$(LIBEXT))

OBJS :=$(patsubst %.c, %$(OBJEXT), $(wildcard *.c))

INCLUDES=\
         $(wildcard $(LIBHOME)/*.h) \
         $(wildcard *.h)

PROGNAME := $(firstword $(patsubst %.pnt,%, $(wildcard *.pnt)))
CLNTNAME := $(firstword $(patsubst %.client,%, $(wildcard *.client)))
LASTNAME := $(firstword $(patsubst %.c,%, $(wildcard *.c)))

FEPLIBS= $(MTSLIB) \
         $(LIBHOME)/fep$(LIBEXT)      \
         $(LIBHOME)/cutil$(LIBEXT)    \
         $(LIBHOME)/snio$(LIBEXT)     \
         $(MTSLIB) \
         $(LIBHOME)/fep$(LIBEXT)      \
         $(LIBHOME)/cutil$(LIBEXT)    \
         $(LIBHOME)/snio$(LIBEXT)
DRVSHELL_MAIN=
DRVSHELL=
TGTEXT=$(EXEEXT)

ifeq ($(OSTYPE),unix)
  FLIBS=$(FEPLIBS)
else
  ifneq ($(FEPLIBS),)
    ifeq ($(OSTYPE),win)
      FLIBS :=$(shell bash -c "cygpath -au $(FEPLIBS)")
    endif
  else
    FLIBS=
  endif
endif

ifeq ($(OBJS),)
  TGTEXT=.def
endif

lib: $(OBJS) zlog$(LIBEXT)

clean:
	@echo
	@echo "Cleaning Directory ..."
	@rm -f $(OBJS)
	@rm -f log$(LIBEXT)
	@rm -f pthread$(OBJEXT)
	@rm -f *.exp
	@rm -f *.lib
	@rm -f *.pch
	@rm -f *.pdb
	@rm -f *.ilk
	@rm -f *.ncb
	@rm -f *.vss
	@rm -f *.scc
	@echo "done."
	@echo

$(OBJS) : %$(OBJEXT) : %.c $(INCLUDES)
	$(CC) $(CFLAGS) -c $<

zlog$(LIBEXT): $(OBJS)
	lib /out:zlog.lib $(OBJS)

