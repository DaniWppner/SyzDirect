# Steps to reach file->remap_file_range instruction through IOCTL$FIDEDUPRANGE in linux v6.15.6 with Syzdirect:

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
Inside the docker container, go to the [examples/vfs_dedupe_file_range_v6.5.15](../../examples/vfs_dedupe_file_range_v6.5.15/) directory.
Then invoke the 'prepare for manual instrument' step of SyzDirect.

```bash
cd /src/SyzDirect/examples/vfs_dedupe_file_range_v6.5.15
python3 /src/SyzDirect/source/syzdirect/Runner/Main.py -dataset ./dedupe_file_range_v6.15.6_dataset.xlsx  -linux-repo-template /src/linux -j 8 -custom-kcov-patch ./kcov_v6.15.6.patch  prepare_for_manual_instrument
```
This will probably clone the linux repo on top of building both Syzkaller and LLVM. **It might take a long while.**


### 3. Instrument the kernel
The previous step will create `workdir/srcs/case_0` on the cwd, which is a clone of the linux repo. 

You want to apply the following patch:
```diff
```

to introduce a `BUG()` that will signal we reached the instruction of interest.

Also apply this patch:
```
```
to signal the fuzzer the area of interest too.

### 4. Execute 'prepare kernel bitcode' step of SyzDirect