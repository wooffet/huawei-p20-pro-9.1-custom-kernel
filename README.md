# Huawei P20 Pro EMUI 9.1 Custom Kernel

This is the custom kernel I am working on to use on my personal P20 Pro, which disables as much root checking as possible and starts SELinux in permissive mode.

The original kernel source code is available at: <https://download-c1.huawei.com/download/downloadCenter?downloadId=101178&version=439689&siteCode=worldwide>

## License

This repository is GPLv2 licensed as the stock kernel build config file is found inside a file that is licensed the same - as far as I'm aware this means it is also GPLv2 licensed.

## Stock Kernel Build Config

- The config used for compiling the kernel is `merge_kirin_defconfig` located at `kernel\arch\arm64\configs\`

## Root Checks

- Some custom root checks from Huawei are present in the kernel, however they can be configured when building so these will be turned off/not have a value set. These have been copied below as they are spread out in multiple config files
- CONFIG_HUAWEI_PROC_CHECK_ROOT will not have a value set to default it to `n` to disable what seems like multiple different function to help check for root
- CONFIG_HW_ROOT_SCAN will not have a value set to default it to `n` to disable the 'Huawei root scanner'
- CONFIG_HUAWEI_EIMA, CONFIG_HUAWEI_EIMA_ACCESS_CONTROL and TEE_ANTIROOT_CLIENT will all be set to `n` as there is no available documentation for these values
    - This means we can't be certain what the default value is, so it is better to just outright disable them
- CONFIG_HW_DOUBLE_FREE_DYNAMIC_CHECK will not have a value set to default it to `n` to remove one communication point with Huawei and prevent some root checking
- CONFIG_HKIP_ATKINFO will not have a value set to default it to `n` to prevent 'attack information' reported by the Huawei Kernel Integrity Protection (HKIP) module to Huawei

## SELinux

- SELinux enforce mode is determined at kernel compile time
    - In stock state, setenforce and other workarounds will not be able to change this once compiled and running
- There are some options for the kernel config file for changing SELinux properties, see the SELinux Kconfig file either below or at `kernel\security\selinux\` in the original source
- SECURITY_SELINUX will be left as `y` to keep it enabled
- SECURITY_SELINUX_DEVELOP will be set as `y` to have the kernel start in permissive mode and should allow for the mode to be changed at runtime
	- Setting at runtime may not be possible if a policy is set to deny this, not sure if this is controllable when building the kernel from source yet
- *SECURITY_SELINUX_DISABLE could be set as `y` to enable SELinux to be disabled at runtime if required, but this seems risky*

### Root Checks Kconfig values

```
No Kconfig values present in open source kernel code for CONFIG_TEE_ANTIROOT_CLIENT, the default value is below:
CONFIG_TEE_ANTIROOT_CLIENT=y

No Kconfig values present in open source kernel code for CONFIG_HUAWEI_EIMA, CONFIG_HUAWEI_ENG_EIMA, CONFIG_HUAWEI_EIMA_ACCESS_CONTROL but found a Huawei security whitepaper that references 'EMUI Integrity Measurement Architecture (EIMA)' and their default values are below:
<https://consumer-img.huawei.com/content/dam/huawei-cbg-site/en/mkt/legal/privacy-policy/EMUI%209.0%20Security%20Technology%20White%20Paper.pdf>
CONFIG_HUAWEI_EIMA=y
# CONFIG_HW_ROOT_SCAN_ENG_DEBUG is not set
CONFIG_HUAWEI_EIMA_ACCESS_CONTROL=y

config HW_ROOT_SCAN
	tristate "enables Huawei root scanner"
	default n
	help
	  This option enables support for Huawei root scanner in kernel space.
	  Huawei root scanner will check if the device is got rooting or not
	  base on five items, kernel code integrity, system call table
	  integrity, SeLinux status, SeLinux hooks status, and root processes.

config HW_ROOT_SCAN_ENG_DEBUG
	bool "Huawei root scanner for engineering mode debug"
	depends on HW_ROOT_SCAN
	default n
	help
	  This option should only be enabled for engineering mode & debug test.
	  In engineering mode, root scanner will be turnoff(default) after init,
	  tester can use supported interface to turn it on, and do debug test.

config HW_DOUBLE_FREE_DYNAMIC_CHECK
	bool "hw double free dynamic check"
	default n
	help
	support double free check function.
	upload the call stack information to.
	hisecd when double free happen.
function description comment
/*
 * upload_double_free_log - upload stack trace infomation when double free happen
 * @s, the kmem_cache happen double free
 * @add_info, the add_info wanted to upload
 * @return: void
 */

config HKIP_ATKINFO
	bool "Upload HKIP attack information"
	depends on HISI_HHEE
	default n
	help
	Upload the attack information reported by HKIP to Huawei server.

config HKIP_ATKINFO_DEBUGFS
	bool "Upload HKIP attack information debugfs"
	depends on HKIP_ATKINFO
	default n
	help
	The debugfs interface for the feature of uploading the attack information.
```

### SELinux Kconfig file

```
config SECURITY_SELINUX
	bool "NSA SELinux Support"
	depends on SECURITY_NETWORK && AUDIT && NET && INET
	select NETWORK_SECMARK
	default n
	help
	  This selects NSA Security-Enhanced Linux (SELinux).
	  You will also need a policy configuration and a labeled filesystem.
	  If you are unsure how to answer this question, answer N.

config SECURITY_SELINUX_BOOTPARAM
	bool "NSA SELinux boot parameter"
	depends on SECURITY_SELINUX
	default n
	help
	  This option adds a kernel parameter 'selinux', which allows SELinux
	  to be disabled at boot.  If this option is selected, SELinux
	  functionality can be disabled with selinux=0 on the kernel
	  command line.  The purpose of this option is to allow a single
	  kernel image to be distributed with SELinux built in, but not
	  necessarily enabled.

	  If you are unsure how to answer this question, answer N.

config SECURITY_SELINUX_BOOTPARAM_VALUE
	int "NSA SELinux boot parameter default value"
	depends on SECURITY_SELINUX_BOOTPARAM
	range 0 1
	default 1
	help
	  This option sets the default value for the kernel parameter
	  'selinux', which allows SELinux to be disabled at boot.  If this
	  option is set to 0 (zero), the SELinux kernel parameter will
	  default to 0, disabling SELinux at bootup.  If this option is
	  set to 1 (one), the SELinux kernel parameter will default to 1,
	  enabling SELinux at bootup.

	  If you are unsure how to answer this question, answer 1.

config SECURITY_SELINUX_DISABLE
	bool "NSA SELinux runtime disable"
	depends on SECURITY_SELINUX
	default n
	help
	  This option enables writing to a selinuxfs node 'disable', which
	  allows SELinux to be disabled at runtime prior to the policy load.
	  SELinux will then remain disabled until the next boot.
	  This option is similar to the selinux=0 boot parameter, but is to
	  support runtime disabling of SELinux, e.g. from /sbin/init, for
	  portability across platforms where boot parameters are difficult
	  to employ.

	  If you are unsure how to answer this question, answer N.

config SECURITY_SELINUX_DEVELOP
	bool "NSA SELinux Development Support"
	depends on SECURITY_SELINUX
	default y
	help
	  This enables the development support option of NSA SELinux,
	  which is useful for experimenting with SELinux and developing
	  policies.  If unsure, say Y.  With this option enabled, the
	  kernel will start in permissive mode (log everything, deny nothing)
	  unless you specify enforcing=1 on the kernel command line.  You
	  can interactively toggle the kernel between enforcing mode and
	  permissive mode (if permitted by the policy) via /selinux/enforce.

config SECURITY_SELINUX_AVC_STATS
	bool "NSA SELinux AVC Statistics"
	depends on SECURITY_SELINUX
	default y
	help
	  This option collects access vector cache statistics to
	  /selinux/avc/cache_stats, which may be monitored via
	  tools such as avcstat.

config SECURITY_SELINUX_CHECKREQPROT_VALUE
	int "NSA SELinux checkreqprot default value"
	depends on SECURITY_SELINUX
	range 0 1
	default 0
	help
	  This option sets the default value for the 'checkreqprot' flag
	  that determines whether SELinux checks the protection requested
	  by the application or the protection that will be applied by the
	  kernel (including any implied execute for read-implies-exec) for
	  mmap and mprotect calls.  If this option is set to 0 (zero),
	  SELinux will default to checking the protection that will be applied
	  by the kernel.  If this option is set to 1 (one), SELinux will
	  default to checking the protection requested by the application.
	  The checkreqprot flag may be changed from the default via the
	  'checkreqprot=' boot parameter.  It may also be changed at runtime
	  via /selinux/checkreqprot if authorized by policy.

	  If you are unsure how to answer this question, answer 0.

config HISI_SELINUX_EBITMAP_RO
        bool "Protect extensible bitmaps used to represent the SE Linux policy DB"
        select HISI_PMALLOC
        default n
        help
          Those extensible bitmaps that are used to store parts of the policy DB are
          protected and marked read-only, when the option is enabled.

config HISI_SELINUX_PROT
        bool "Enable protection for SELinux policy DB"
        select HISI_PMALLOC
        select HISI_RO_LSM_HOOKS
        select HISI_SELINUX_EBITMAP_RO
        default n
        help
          Adds hardening to various data structures, that are both constant
          and semi-constant in nature.
          Also loads into a special memory pool the data structures created
          when receiveing in input the policy description.
          The pool is write-protected, once loading is complete.
```
