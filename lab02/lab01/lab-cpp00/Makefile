#MAC_OPTS=-isystem /opt/homebrew/include -I/opt/homebrew/include -L/opt/homebrew/lib
#ARCH = -arch arm64

CC = $(CLANG_PREFIX)clang++
LAB_OPTS = -I./src src/lib.cpp
C_OPTS = $(MAC_OPTS) -fsanitize=address -std=c++11 -g -Wall -Wextra -Werror -Wformat-security -Wfloat-equal -Wshadow -Wconversion -Wlogical-not-parentheses -Wnull-dereference -Wno-unused-variable -Werror=vla $(LAB_OPTS)

clean:
	rm -rf dist
prep:
	mkdir dist
compile: main.bin test.bin

main.bin: src/main.cpp
	$(CC) $(C_OPTS) $< -o ./dist/$@
test.bin: test/test.cpp
	$(CC) $(C_OPTS) $< $(ARCH) -fprofile-instr-generate -fcoverage-mapping -o ./dist/$@ -lgtest -lgtest_main
test: clean prep compile
	LLVM_PROFILE_FILE="dist/test.profraw" ./dist/test.bin
	$(CLANG_PREFIX)llvm-profdata merge -sparse dist/test.profraw -o dist/test.profdata
	$(CLANG_PREFIX)llvm-cov report dist/test.bin -instr-profile=dist/test.profdata src/*.cpp
	$(CLANG_PREFIX)llvm-cov show dist/test.bin -instr-profile=dist/test.profdata src/*.cpp --format html > dist/coverage.html
run: clean prep compile
	./dist/main.bin
check:
	clang-format --verbose -dry-run --Werror src/* test/*
	$(CLANG_PREFIX)clang-tidy src/*.cpp  -checks=-readability-uppercase-literal-suffix,-readability-magic-numbers,-clang-analyzer-deadcode.DeadStores,-clang-analyzer-security.insecureAPI.rand
	cppcheck --enable=all --inconclusive --addon=cert --error-exitcode=1  -v --check-config --suppress=*:*/test.cpp  .
	rm -rf src/*.dump test/*.dump
leak-check: clean prep compile
	ASAN_OPTIONS=detect_leaks=1 LSAN_OPTIONS=suppressions=leaks_suppr.txt dist/main.bin
	#ASAN_OPTIONS=detect_leaks=1 LSAN_OPTIONS=suppressions=leaks_suppr.txt LLVM_PROFILE_FILE="dist/test.profraw" dist/test.bin
all: clean prep compile check
