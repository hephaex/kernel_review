# The Linux Kernel review for ARMv7 3.13.0 (exynos 5420)
All of this repository are written by hephaex@gmail.com.

Community name: IAMROOT.ORG ARM kernel study 10th C team
Target Soc    : Samsung Exynos 5420 (ARMv7 A7&A15)
Kernel version: Linux kernel 3.13.x
 - before start_kernel(): 3.10.x
 - start_kernel()->mm_init: 3.13.x 

# The history of Linux kernel study
* 68th (2014/08/30) week study : [68차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_68.md)
 - sched_init()
 - rq 설정 (for_each_possible_cpu(i))
* 67th (2014/08/23) week study : [67차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_67.md)
 - mm_init() 복습
 - slub()　복습
* 66th (2014/08/16) week study : [66차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_66.md)
 - mm_init() 복습;
 - buddy 까지 복습 (mem_init())
* 65th (2014/08/09) week study : [65차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_65.md)
 - start_kernel()-> mm_init()-> vmalloc_init();
 - vmlist에 등록된 vm struct 들을 slab으로 이관하고 RB Tree로 구성
* 64th (2014/07/26) week study : [64차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_64.md)
 - start_kernel()-> mm_init()-> kmem_cache_init()
 - start_kernel()-> mm_init()-> percpu_init_late()
 - start_kernel()-> mm_init()-> pgtable_cache_init()
* 63th (2014/07/19) week study : [63차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_63.md)
 - mm_init()->kmem_cache_init()->bootstrab(&boot_kmem_cache_node) 
* 62th (2014/07/12) week study : [62차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_62.md)
 - mm_init()->kmem_cache_init()->bootstrab(&boot_kmem_cache) 
* 61th (2014/07/05) week study : [61차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_61.md)
* 60th (2014/06/28) week study : [60차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_60.md)
* 59th (2014/06/21) week study : [59차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_59.md)
* 58th (2014/06/14) week study : [58차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_58.md)
* 57th (2014/06/07) week study : [57차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_57.md)
 - start_kernel()->mm_init()->kmem_cache_init()->create_boot_cache()
 - slab_state는 slab이 어느정도 활성화 되었는지를 나타낸다.
 - 지금까지 // slab_state: DOWN 에서 분석을 했고,
 - 이제는 slab_state = PARTIAL; 로 바뀌어 분석을 한다. 
 - 여기서  // slab_state의 의미는  slab을 초기화한 단계를 의미한다.
 - slab_stat = PARTIAL은  kmem_cache_node만 사용이 가능을 의미한다.
 - 계속해서 slab_stat = PARTIAL로 해서 create_boot_cache를 다시 실행한다. 
* 56th (2014/05/31) week study : [56차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_56.md)
 - start_kernel()->mm_init()->kmem_cache_init()->create_boot_cache()
 - init_kmem_cache_nodes는 slab으로 사용할 page를 할당받아 설정값(slab_cache, flags, freelist, inuse, frozen)을 바꿔준다.
 - 이후 할당받은 slab object를 kmem_cache_node로 사용하며,
 - kmem_cache_node의 맴버 속성을 초기화합니다. 
 - 초기화되는 맴버속성은 (nr_partial, list_lock, slabs, full)가 있습니다.
* 55th (2014/05/24) week study : [55차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_55.md)
 - start_kernel()->mm_init()->kmem_cache_init()->create_boot_cache()
 - new_slab()
* 54th (2014/05/17) week study : [54차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_54.md) 
 - buddy할당자에 이어서 kmem 할당자 (Slub)을 분석중입니다. 
 - mem_init()->kmem_cache_init()->create_boot_cache()->__kmem_cache_create()
  ->kmem_cache_open()->init_kmem_cache_nodes->early_kmem_cache_node_alloc()
  ->new_slab()->allocate_slab()->alloc_slab_page()->alloc_pages_exact_node()
  ->__alloc_pages()->__alloc_pages_nodemask();
 - 커널 버전을 3.13.11로 바꿨습니다.
* 53th (2014/05/10) week study : [53차 분석](https://github.com/hephaex/kernel_review/blob/master/a10c_53.md)  
...

* 11th (2012-07-06) week study : [11차 분석](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=174738) 18+2명
 - arch/arm/boot/compressed/head.S 분석
 - _setup_mmu 종료
* 10th (2012-06-29) week study: [10차 분석](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=174738) 22명
 - arch/arm/boot/compressed/head.S 분석
 - _setup_mmu 진입직전
* 09th (2012-06-22) week study: [09차 분석](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=171562) 25명
 - Arm System Developer's Guide (Ch.14 ~ 끝)
 - arch/arm/boot/compressed/head.S 분석
 - call_cache_fn 진입직전
* 08th (2012-06-15) week study: [08차 분석] 21명
 - Arm System Developer's Guide (Ch.09 ~ Ch.14.4 페이지 테이블)
* 07th (2012-06-08) week study: [07차 분석] 20명
 - Arm System Developer's Guide (시작 ~ Ch.09 인터럽트 처리방법)
* 06th (2012-06-01) week study: [06차 분석]
 - ARM v7 아키텍쳐 세미나
* 05th (2012-05-25) week study: [05차 분석]
 - ARM v7 아키텍쳐 세미나
* 04th (2012-05-18) week study: [04차 분석] 28명+1 (백창우님)
 - Arm System Developer's Guide (pt자료)
* 03th (2012-05-11) week study: [03차 분석] 22명
 - 리눅스 커널 내부구조 (p.150 ~ 끝)
* 02th (2012-05-04) week study: [02차 분석] 27명
 - 리눅스 커널 내부구조 (p. 88~ p.150)
* 01th (2012-04-28) week study: [01차 분석] 34명
 - 리눅스 커널 내부구조 (처음  ~ p. 88)
