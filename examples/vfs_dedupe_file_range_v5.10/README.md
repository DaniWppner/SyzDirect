# Steps to reach file->remap_file_range instruction through IOCTL$FIDEDUPRANGE in linux v5.10 with Syzdirect:

### 1. Launch docker environment.
Edit the paths on [the docker-compose file](../../docker-environ/docker-compose.yml) if you haven't yet. Then run:
```bash
cd SyzDirect/docker-environ 
docker compose up
```
This will launch the docker container and keep printing logs to the running terminal. You can stop it with Ctrl+C.

In another terminal, run:
```bash
docker exec -it syzdirect-environment bash
```
To get into the container. The steps to follow assume you're working inside the docker terminal. 

### 2. Execute 'prepare for manual instrument' step of SyzDirect
Inside the docker container, go to the [examples/vfs_dedupe_file_range_v5.10](../../examples/vfs_dedupe_file_range_v5.10/) directory.
Then invoke the 'prepare for manual instrument' step of SyzDirect.

```bash
cd /src/SyzDirect/examples/vfs_dedupe_file_range_v5.10
python3 /src/SyzDirect/source/syzdirect/Runner/Main.py -dataset ./dedupe_file_range_v5.10_dataset.xlsx  -linux-repo-template /src/linux -j 8 -custom-kcov-patch ./kcov_v5.10.patch  prepare_for_manual_instrument
```
This will probably clone the linux repo on top of building both Syzkaller and LLVM. **It might take a long while.**


### 3. Instrument the kernel
The previous step will create `workdir/srcs/case_0` on the cwd, which is a clone of the linux repo. 

You want to apply the following patch:
```diff
diff --git a/fs/remap_range.c b/fs/remap_range.c
index 26afbbbfb10c..fbf76fefe438 100644
--- a/fs/remap_range.c
+++ b/fs/remap_range.c
@@ -21,6 +21,8 @@
 #include <linux/uaccess.h>
 #include <asm/unistd.h>
 
+#include <linux/kcov.h>
+
 /*
  * Performs necessary checks before doing a clone.
  *
@@ -480,6 +482,9 @@ loff_t vfs_dedupe_file_range_one(struct file *src_file, loff_t src_pos,
 		goto out_drop_write;
 	}
 
+	kcov_mark_block(0);
+	pr_err("Reachable: file->f_op->dedupe_file_range");
+	BUG();
 	ret = dst_file->f_op->remap_file_range(src_file, src_pos, dst_file,
 			dst_pos, len, remap_flags | REMAP_FILE_DEDUP);
 out_drop_write:

```

to introduce a `BUG()` that will signal we reached the instruction of interest, and to signal the directed fuzzer the area of interest as well.

### 4. Execute 'prepare kernel bitcode' step of SyzDirect


```bash
python3 /src/SyzDirect/source/syzdirect/Runner/Main.py -dataset ./dedupe_file_range_v5.10_dataset.xlsx -j 8 prepare_kernel_bitcode
```

### 5. Execute 'analyze kernel syscall' step of Syzdirect

### 6. Execute 'extract_syscall_entry' step of Syzdirect

### 7. Execute 'instrument_kernel_with_distance' step of Syzdirect

### 8. Execute 'fuzz' step of Syzdirect
