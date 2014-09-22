CC = gcc
STD_CFLAGS = -Wall -std=gnu99 -c -g -O3

# Enable ARM-specific options only on ARM, and compilation of the app only on ARM
UNAME := $(shell uname -m)

ifeq ($(UNAME), armv6l)
	CFLAGS = $(STD_CFLAGS) -march=armv6 -mtune=arm1176jzf-s -mfloat-abi=hard -mfpu=vfp -ffast-math

app: pi_fm_rds.o fm_mpx.o
	$(CC) -o pi_fm_dma pi_fm_rds.o fm_mpx.o -lm -lsndfile

else
	CFLAGS = $(STD_CFLAGS)
endif

pi_fm_rds.o: pi_fm_rds.c fm_mpx.h
	$(CC) $(CFLAGS) pi_fm_rds.c

fm_mpx.o: fm_mpx.c fm_mpx.h
	$(CC) $(CFLAGS) fm_mpx.c

clean:
	rm *.o
