include ../../config.mk

CFLAGS+=$(OPENMPFLAG) -O3 -I../arch -I../common -I../../include -DMAXBUTWIDTH=$(MAXBUTWIDTH)

ifeq ($(COMPILER), icc)
ISALISTSP=1_purecsp 4_sse2sp 1_avxsp 1_avx2sp 1_avx512fsp
ISALISTDP=1_purecdp 4_sse2dp 1_avxdp 1_avx2dp 1_avx512fdp
ISALISTLD=1_purecld
ISALISTQP=1_purecqp
else ifeq ($(X86ARCH),1)

ISALISTDP=1_vecextdp 4_sse2dp 1_avxdp
ISALISTSP=1_vecextsp 4_sse2sp 1_avxsp

ifeq ($(ENABLEAVX2), 1)
ISALISTDP+=1_avx2dp
ISALISTSP+=1_avx2sp
endif

ifeq ($(ENABLEAVX512F), 1)
ISALISTDP+=1_avx512fdp
ISALISTSP+=1_avx512fsp
endif

else ifeq ($(ARCH), arm)
ISALISTDP=1_vecextdp
ISALISTSP=1_vecextsp 1_neon32sp
else ifeq ($(ARCH), aarch64)
ISALISTDP=1_vecextdp 1_advsimddp
ISALISTSP=1_vecextsp 1_advsimdsp
else
ISALISTDP=1_vecextdp
ISALISTSP=1_vecextsp
#ISALISTDP=1_vecextdp 5_vecextdp
#ISALISTSP=1_vecextsp 5_vecextsp
endif

ifneq ($(COMPILER), icc)
ifeq ($(ENABLEFLOAT80), 1)
ISALISTLD=1_vecextld
endif
ifeq ($(ENABLEFLOAT128), 1)
ISALISTQP=1_vecextqp
endif
endif # ifneq ($(COMPILER), icc)

.PHONY: all
all : dft.a

.PHONY: clean
clean :
	rm -f *.o *.s *~ *.exe *.out cachegrind.out.* *.lock
	rm -rf *.dSYM *.obj *.lib *.dll *.exp *.exe
	rm -f a.out mkunroll mkdispatch dispatchdp.h dispatchsp.h dispatchld.h dispatchqp.h dispatchparam.h dft.a
	rm -f unroll_*.c

ifeq ($(ENABLEFLOAT80), 1)
ifeq ($(ENABLEFLOAT128), 1)
dft.a : mkunroll dftdp.o dftsp.o dftld.o dftqp.o dftcommon.o
	./mkunroll double $(ISALISTDP)
	./mkunroll float $(ISALISTSP)
	./mkunroll longdouble $(ISALISTLD)
	./mkunroll Sleef_quad $(ISALISTQP)
	+"$(MAKE)" -f Makefile.unroll CFLAGS="$(CFLAGS) $(SHAREDFLAGS)" OBJ="`echo unroll*.c | sed -e 's/\.c/\.o/g'`"
	$(FLOCK) dft.a.lock -c 'ar rcs dft.a `echo unroll*.o` dftdp.o dftsp.o dftld.o dftqp.o dftcommon.o'
else # ifdef ENABLEFLOAT128
dft.a : mkunroll dftdp.o dftsp.o dftld.o dftcommon.o
	./mkunroll double $(ISALISTDP)
	./mkunroll float $(ISALISTSP)
	./mkunroll longdouble $(ISALISTLD)
	+"$(MAKE)" -f Makefile.unroll CFLAGS="$(CFLAGS) $(SHAREDFLAGS)" OBJ="`echo unroll*.c | sed -e 's/\.c/\.o/g'`"
	$(FLOCK) dft.a.lock -c 'ar rcs dft.a `echo unroll*.o` dftdp.o dftsp.o dftld.o dftcommon.o'
endif # ifdef ENABLEFLOAT128
else # ifdef ENABLEFLOAT80
dft.a : mkunroll dftdp.o dftsp.o dftcommon.o
	./mkunroll double $(ISALISTDP)
	./mkunroll float $(ISALISTSP)
	+"$(MAKE)" -f Makefile.unroll CFLAGS="$(CFLAGS) $(SHAREDFLAGS)" OBJ="`echo unroll*.c | sed -e 's/\.c/\.o/g'`"
	$(FLOCK) dft.a.lock -c 'ar rcs dft.a `echo unroll*.o` dftdp.o dftsp.o dftcommon.o'
endif # ifdef ENABLEFLOAT80

dftdp.o : dft.c dispatchdp.h ../../include/sleef.h
	$(CC) $(CFLAGS) $(SHAREDFLAGS) $(OPTFLAGS) -DBASETYPEID=1 dft.c -c -o dftdp.o

dftsp.o : dft.c dispatchsp.h ../../include/sleef.h
	$(CC) $(CFLAGS) $(SHAREDFLAGS) $(OPTFLAGS) -DBASETYPEID=2 dft.c -c -o dftsp.o

dftld.o : dft.c dispatchld.h ../../include/sleef.h
	$(CC) $(CFLAGS) $(SHAREDFLAGS) $(OPTFLAGS) -DBASETYPEID=3 dft.c -c -o dftld.o

dftqp.o : dft.c dispatchqp.h ../../include/sleef.h
	$(CC) $(CFLAGS) $(SHAREDFLAGS) $(OPTFLAGS) -DBASETYPEID=4 dft.c -c -o dftqp.o

dftcommon.o : dftcommon.c dispatchparam.h ../../include/sleef.h
	$(CC) $(CFLAGS) $(SHAREDFLAGS) $(OPTFLAGS) dftcommon.c -c -o dftcommon.o

mkdispatch : mkdispatch.c
	$(CC) $(CFLAGS) -Wall -Wno-unused -Wno-attributes mkdispatch.c -o mkdispatch

mkunroll : mkunroll.c
	$(CC) $(CFLAGS) -Wall -Wno-unused -Wno-attributes mkunroll.c -o mkunroll

dispatchdp.h : mkdispatch
	./mkdispatch double $(UNROLLMAX) $(UNROLLMAX2) $(MAXBUTWIDTH) $(ISALISTDP) > dispatchdp.h

dispatchsp.h : mkdispatch
	./mkdispatch float $(UNROLLMAX) $(UNROLLMAX2) $(MAXBUTWIDTH) $(ISALISTSP) > dispatchsp.h

dispatchld.h : mkdispatch
	./mkdispatch longdouble $(UNROLLMAX) $(UNROLLMAX2) $(MAXBUTWIDTH) $(ISALISTLD) > dispatchld.h

dispatchqp.h : mkdispatch
	./mkdispatch Sleef_quad $(UNROLLMAX) $(UNROLLMAX2) $(MAXBUTWIDTH) $(ISALISTQP) > dispatchqp.h

dispatchparam.h : mkdispatch
	./mkdispatch paramonly $(UNROLLMAX) $(UNROLLMAX2) $(MAXBUTWIDTH) $(ISALISTDP) > dispatchparam.h

../../include/sleef.h :
	+"$(MAKE)" --directory=../../include sleef.h
