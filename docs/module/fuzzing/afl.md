# AFL

## 种子同步问题

如果我有一个好的种子，想要加入一个正在运行的AFL实例中，怎么办

### distributed mode

如果AFL以并行模式运行（-M / -S），比较好办

#### AFL并行模式同步机制

关键数据sync_dir和sync_id

```c
EXP_ST u8 *sync_dir,                  /* Synchronization directory        */
          *sync_id,                   /* Fuzzer ID                        */
```

sync_id就是 -M / -S 后面的参数

```c
      case 'M': { /* master sync ID */

          u8* c;

          if (sync_id) FATAL("Multiple -S or -M options not supported");
          sync_id = ck_strdup(optarg);
```

sync_dir的值等于-o 后面的参数，也就是说并行模式下out目录变成sync_dir, 真正的out目录变成 sync_dir/sync_id

```c
static void fix_up_sync(void) {

  //...
  
  x = alloc_printf("%s/%s", out_dir, sync_id);

  sync_dir = out_dir;
  out_dir  = x;
  
  //...
```

执行同步的函数 `sync_fuzzers`

```c
static void sync_fuzzers(char** argv) {

  DIR* sd;
  struct dirent* sd_ent;
  u32 sync_cnt = 0;

  sd = opendir(sync_dir);
  if (!sd) PFATAL("Unable to open '%s'", sync_dir);

  stage_max = stage_cur = 0;
  cur_depth = 0;

  /* Look at the entries created for every other fuzzer in the sync directory. */

  while ((sd_ent = readdir(sd))) {

    static u8 stage_tmp[128];

    DIR* qd;
    struct dirent* qd_ent;
    u8 *qd_path, *qd_synced_path;
    u32 min_accept = 0, next_min_accept;

    s32 id_fd;

    /* Skip dot files and our own output directory. */

    if (sd_ent->d_name[0] == '.' || !strcmp(sync_id, sd_ent->d_name)) continue;

    /* Skip anything that doesn't have a queue/ subdirectory. */
    /* 跳过没有queue这个子目录的文件夹 */

    qd_path = alloc_printf("%s/%s/queue", sync_dir, sd_ent->d_name);

    if (!(qd = opendir(qd_path))) {
      ck_free(qd_path);
      continue;
    }

    /* Retrieve the ID of the last seen test case. */

    qd_synced_path = alloc_printf("%s/.synced/%s", out_dir, sd_ent->d_name);

    id_fd = open(qd_synced_path, O_RDWR | O_CREAT, 0600);

    if (id_fd < 0) PFATAL("Unable to create '%s'", qd_synced_path);

    if (read(id_fd, &min_accept, sizeof(u32)) > 0)
      lseek(id_fd, 0, SEEK_SET);
    /* 获取最小可接受id值，其值等于上次同步的种子id + 1
    next_min_accept = min_accept;

    /* Show stats */

    sprintf(stage_tmp, "sync %u", ++sync_cnt);
    stage_name = stage_tmp;
    stage_cur  = 0;
    stage_max  = 0;

    /* For every file queued by this fuzzer, parse ID and see if we have looked at
       it before; exec a test case if not. */

    while ((qd_ent = readdir(qd))) { /* 打开另一个fuzz实例的out目录下的queue文件夹

      u8* path;
      s32 fd;
      struct stat st;
      /* 排除 '.'开头的文件，排除非id:XXXXXX开头的文件，排除XXXXXX小于上次记录的id的文件 */
      if (qd_ent->d_name[0] == '.' ||
          sscanf(qd_ent->d_name, CASE_PREFIX "%06u", &syncing_case) != 1 ||
          syncing_case < min_accept) continue;

      /* OK, sounds like a new one. Let's give it a try. */

      if (syncing_case >= next_min_accept)
        next_min_accept = syncing_case + 1;

      path = alloc_printf("%s/%s", qd_path, qd_ent->d_name);

      /* Allow this to fail in case the other fuzzer is resuming or so... */

      fd = open(path, O_RDONLY);

      if (fd < 0) {
         ck_free(path);
         continue;
      }

      if (fstat(fd, &st)) PFATAL("fstat() failed");

      /* Ignore zero-sized or oversized files. */

      if (st.st_size && st.st_size <= MAX_FILE) {

        u8  fault;
        u8* mem = mmap(0, st.st_size, PROT_READ, MAP_PRIVATE, fd, 0);

        if (mem == MAP_FAILED) PFATAL("Unable to mmap '%s'", path);

        /* See what happens. We rely on save_if_interesting() to catch major
           errors and save the test case. */

        write_to_testcase(mem, st.st_size);

        fault = run_target(argv, exec_tmout);

        if (stop_soon) return;

        syncing_party = sd_ent->d_name;
        queued_imported += save_if_interesting(argv, mem, st.st_size, fault);
        syncing_party = 0;

        munmap(mem, st.st_size);

        if (!(stage_cur++ % stats_update_freq)) show_stats();

      }

      ck_free(path);
      close(fd);

    }

    ck_write(id_fd, &next_min_accept, sizeof(u32), qd_synced_path);

    close(id_fd);
    closedir(qd);
    ck_free(qd_path);
    ck_free(qd_synced_path);

  }  

  closedir(sd);

}
```

大致流程

```shell
for 目录 in 打开sync_dir目录并获取目录下的所有目录
do
  if 目录没有queue这个子目录 -o 目录名称等于当前sync_id, 即不同步自己; then 跳过; fi
  获取最小可接受id值，其值等于上次同步的种子id + 1
  for seed in 打开queue目录下的所有文件
    do
      if '.'开头的文件 -o 非id:XXXXXX开头的文件 -o id:XXXXXX小于最小可接受id值; then 跳过; fi
      更新最小可接受id值，其值等于当前id + 1
      save_if_interesting(argv, mem, st.st_size, fault);
    done
done
```

save_if_interesting 流程

在非crash_mode下，同步的种子如果超时或者崩溃，是不会加入队列的。如果bitmap有更新，是会计入hangs或者crashes

在crash_mode下，同步的种子崩溃，加入队列，如果bitmap有更新计入crashes。超时处理与非crash_mode相同

```c
static u8 save_if_interesting(char** argv, void* mem, u32 len, u8 fault) {
  // ...
  if (fault == crash_mode) {
    /* crash_mode 默认为0，也就是FAULT_NONE, '-C' 指定crash_mode为FAULT_CRASH */
    /* bitmap不更新直接返回 */
    // ...
    fn = alloc_printf("%s/queue/id:%06u,%s", out_dir, queued_paths,
                      describe_op(hnb));
    // ...
    keeping = 1;
  }

  switch (fault) {

    case FAULT_TMOUT:
      /* 如果没有更新或者大于上限直接返回 */
      // ..
      fn = alloc_printf("%s/hangs/id:%06llu,%s", out_dir,
                        unique_hangs, describe_op(0));
      // ...

    case FAULT_CRASH:
      /* 如果没有更新或者大于上限直接返回 */
      // ...
      fn = alloc_printf("%s/crashes/id:%06llu,sig:%02u,%s", out_dir,
                        unique_crashes, kill_signal, describe_op(0));
      // ...
  }

  /* If we're here, we apparently want to save the crash or hang
     test case, too. */
  // ...
  return keeping;
}
```

AFL_HANG_TMOUT
