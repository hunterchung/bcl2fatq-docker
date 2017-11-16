# Bcl2fastq-1.8.4

## Quick Start

Replace `<run folder>` and `<output folder>` with the real directory names on host machine. Normally the name of `<run folder>` may look something like `100723_EAS346_0188_FC626BWAAXX`. Make sure the user that runs `docker run` command on host machine has the read permission to `<run folder>` and the read/write permission to `<output folder>`.

```bash
docker run -d --name bcl2fastq \
    -u $( id -u $USER ):$( id -g $USER ) \
    -v <run folder>:/mnt/run \
    -v <output folder>:/mnt/output \
    zymoresearch/bcl2fastq
```

This would run the following `bcl2fastq` command in container.

```bash
sh -c /usr/local/bin/configureBclToFastq.pl \
    --input-dir /mnt/run/Data/Intensities/BaseCalls/ \
    --output-dir /mnt/output/Unaligned \
    --fastq-cluster-count 0 \
    --mismatches 1 \
    --no-eamss \
    --with-failed-reads && \
make -j 4 -C /mnt/output/Unaligned/
```


## Modify Bcl2fastq Command

### Modify the extent of parallelization

Override the default setting (default: 4) by passing an environment variable `CPU_NUM` to `docker run` command. This equivalent to running `make -j $CPU_NUM` command for bcl conversion and demultiplexing.

```bash
docker run -d --name bcl2fastq \
    -e "CPU_NUM=8" \
    -u $( id -u $USER ):$( id -g $USER ) \
    -v <run folder>:/mnt/run \
    -v <output folder>:/mnt/output \
    zymoresearch/bcl2fastq
```

### Modify the number of mismatches allowed for each index read

Override the default setting (default: 1) by passing an environment variable `MISMATCHES` to `docker run` command. This can dynamically modify the `--mismatches` option of `configureBclToFastq.pl` command executed in container.

```bash
docker run -d --name bcl2fastq \
    -e "MISMATCHES=0" \
    -u $( id -u $USER ):$( id -g $USER ) \
    -v <run folder>:/mnt/run \
    -v <output folder>:/mnt/output \
    zymoresearch/bcl2fastq
```

### Custom Bcl2fastq command

You can run your own `bcl2fastq` command by specifying arguments to `docker run`. Please notice that a complete command  for bcl conversion and demultiplexing should consist of both `configureBclToFastq.pl` and `make`. For example:

```bash
docker run -d --name bcl2fastq \
    -u $( id -u $USER ):$( id -g $USER ) \
    -v <run folder>:/mnt/run \
    -v <output folder>:/mnt/output \
    zymoresearch/bcl2fastq \
    configureBclToFastq.pl \
      --input-dir /mnt/run/Data/Intensities/BaseCalls \
      --output-dir /mnt/output \
      --no-eamss && \
    make -j 4 -C /mnt/output
```

**Alternatively**, you can run an interactive shell first in the image.

```bash
docker run -it --name bcl2fastq\
    -u $( id -u $USER ):$( id -g $USER ) \
    -v <run folder>:/mnt/run \
    -v <output folder>:/mnt/output \
    zymoresearch/bcl2fastq bash
```

And then execute `bcl2fastq` command in the shell. For example:

```bash
/usr/local/bin/configureBclToFastq.pl \
    --input-dir /mnt/run/Data/Intensities/BaseCalls \
    --output-dir /mnt/output \
    --no-eamss
make -j 4 -C /mnt/output
```

Use `ctrl-p + ctrl-q` to detach the tty without exiting the shell.


## Note

* Run `docker logs <container name>` to fetch the logs of a container, and run `docker logs -f <container name>` to follow container log output real-time.
