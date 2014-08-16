#IAMROOT.ORG Kernel스터디10차(ARM)

#HISTORY
  - 65th (2014/08/09) week study : [64차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_65.md)
    - start_kernel()-> mm_init()-> vmalloc_init();
	- vmlist에 등록된 vm struct 들을 slab으로 이관하고 RB Tree로 구성
  - 64th (2014/07/26) week study : [64차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_64.md)
    - start_kernel()-> mm_init()-> kmem_cache_init()
    - start_kernel()-> mm_init()-> percpu_init_late()
    - start_kernel()-> mm_init()-> pgtable_cache_init()
  - 63th (2014/07/19) week study : [63차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_63.md)
    - mm_init()->kmem_cache_init()->bootstrab(&boot_kmem_cache_node) 
  - 62th (2014/07/12) week study : [62차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_62.md)
    - mm_init()->kmem_cache_init()->bootstrab(&boot_kmem_cache) 
  - 61th (2014/07/05) week study : [61차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_61.md)
  - 60th (2014/06/28) week study : [60차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_60.md)
  - 59th (2014/06/21) week study : [59차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_59.md)
  - 58th (2014/06/14) week study : [58차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_58.md)
  - 57th (2014/06/07) week study : [57차 스터디](https://github.com/hephaex/a10c_review/blob/master/a10c_57.md)
...

  - 11th (2012-07-06) week study : [11차 스터디](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=174738) 18+2명
    - arch/arm/boot/compressed/head.S 분석
    - _setup_mmu 종료
  - 10th (2012-06-29) week study: [10차 스터디](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=174738) 22명
    - arch/arm/boot/compressed/head.S 분석
    - _setup_mmu 진입직전
  - 09th (2012-06-22) week study: [09차 스터디](http://www.iamroot.org/xe/index.php?mid=Kernel_10_ARM&category=172676&page=6&document_srl=171562) 25명
    - arch/arm/boot/compressed/head.S 분석
	- call_cache_fn 진입직전
    - Arm System Developer's Guide (Ch.14 ~ 끝)
  - 08th (2012-06-15) week study: [08차 스터디]	21명
    - Arm System Developer's Guide (Ch.09 ~ Ch.14.4 페이지 테이블)
  - 07th (2012-06-08) week study: [07차 스터디]	20명
    - Arm System Developer's Guide (시작 ~ Ch.09 인터럽트 처리방법)
  - 06th (2012-06-01) week study: [06차 스터디]
    - ARM v7 아키텍쳐 세미나
  - 05th (2012-05-25) week study: [05차 스터디]
    - ARM v7 아키텍쳐 세미나
  - 04th (2012-05-18) week study: [04차 스터디]	28명+1 (백창우님)
    - Arm System Developer's Guide (pt자료)
  - 03th (2012-05-11) week study: [03차 스터디]	22명
    - 리눅스 커널 내부구조 (p.150 ~ 끝)
  - 02th (2012-05-04) week study: [02차 스터디] 27명
     - 리눅스 커널 내부구조 (p. 88~ p.150)
  - 01th (2012-04-28) week study: [01차 스터디] 34명
    - 리눅스 커널 내부구조 (처음  ~ p. 88)
