# SPDX-License-Identifier: GPL-2.0
#
# Kbuild for top-level directory of the kernel
# This file takes care of the following:
# 1) Generate bounds.h
# 2) Generate timeconst.h
# 3) Generate asm-offsets.h (may need bounds.h and timeconst.h)
# 4) Check for missing system calls
# 5) Generate constants.py (may need bounds.h)

#####
# 1) Generate bounds.h

bounds-file := include/generated/bounds.h

always  := $(bounds-file)
targets := kernel/bounds.s

# We use internal kbuild rules to avoid the "is up to date" message from make
kernel/bounds.s: kernel/bounds.c FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,cc_s_c)

$(obj)/$(bounds-file): kernel/bounds.s FORCE
	$(call filechk,offsets,__LINUX_BOUNDS_H__)

#####
# 2) Generate timeconst.h

timeconst-file := include/generated/timeconst.h

targets += $(timeconst-file)

quiet_cmd_gentimeconst = GEN     $@
define cmd_gentimeconst
	(echo $(CONFIG_HZ) | bc -q $< ) > $@
endef
define filechk_gentimeconst
	(echo $(CONFIG_HZ) | bc -q $< )
endef

$(obj)/$(timeconst-file): kernel/time/timeconst.bc FORCE
	$(call filechk,gentimeconst)

#####
# 3) Generate asm-offsets.h
#

offsets-file := include/generated/asm-offsets.h

cppflags-$(CONFIG_HIF_CPU_PERF_AFFINE_MASK) += -DHIF_CPU_PERF_AFFINE_MASK
cppflags-$(CONFIG_SMMU_S1_UNMAP) += -DCONFIG_SMMU_S1_UNMAP
cppflags-$(CONFIG_GENERIC_SHADOW_REGISTER_ACCESS_ENABLE) += -DGENERIC_SHADOW_REGISTER_ACCESS_ENABLE
cppflags-$(CONFIG_IPA_SET_RESET_TX_DB_PA) += -DIPA_SET_RESET_TX_DB_PA
cppflags-$(CONFIG_DUMP_REO_QUEUE_INFO_IN_DDR) += -DDUMP_REO_QUEUE_INFO_IN_DDR

# We use internal kbuild rules to avoid the "is up to date" message from make
arch/$(SRCARCH)/kernel/asm-offsets.s: arch/$(SRCARCH)/kernel/asm-offsets.c \
                                      $(obj)/$(timeconst-file) $(obj)/$(bounds-file) FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,cc_s_c)

$(obj)/$(offsets-file): arch/$(SRCARCH)/kernel/asm-offsets.s FORCE
	$(call filechk,offsets,__ASM_OFFSETS_H__)

#####
# 4) Check for missing system calls
#

always += missing-syscalls
targets += missing-syscalls

quiet_cmd_syscalls = CALL    $<
      cmd_syscalls = $(CONFIG_SHELL) $< $(CC) $(c_flags) $(missing_syscalls_flags)

missing-syscalls: scripts/checksyscalls.sh $(offsets-file) FORCE
	$(call cmd,syscalls)

#####
# 5) Generate constants for Python GDB integration
#

extra-$(CONFIG_GDB_SCRIPTS) += build_constants_py

build_constants_py: $(obj)/$(timeconst-file) $(obj)/$(bounds-file)
	@$(MAKE) $(build)=scripts/gdb/linux $@

# Keep these three files during make clean
no-clean-files := $(bounds-file) $(offsets-file) $(timeconst-file)
