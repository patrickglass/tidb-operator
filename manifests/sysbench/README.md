
# sysbench

## Build Image

```shell
# New profile to allow building multi-arch images in parallel
docker buildx create --use
# Build the image for both amd64 and arm64 (Mac M1) 
docker buildx build --platform linux/amd64,linux/arm64 -t patrickglass526/sysbench:latest --push .
```

## configuration

Modify the configuration in `base/sysbench.conf` and `base/options.conf` 

```
export MYSQL_HOST=127.0.0.1
export MYSQL_PS1="tidb> "
export MYSQL_TCP_PORT=4000
export MYSQL_PWD='z06wXmC45pR9I!gB9X0kjzQDTvdbFo'
```


## execute sysbench job

```shell
kubectx "${ENVIRONMENT}-${SHARD_NAME}"
kubens tidb

func run_sysbench() {
    local jobname=$1
    local timeout="${2:-1200}"
    local jobfolder=$(echo "$1" | tr '-' '_')
    kubectl apply -k "${jobfolder}" && \
        kubectl wait --for=condition=complete job/sysbench-${jobname} --timeout="${timeout}s"
}

func sysbench_summary() {
    local jobname=$1
    echo "INFO: Run Summary $jobname"
    kubectl logs "job/sysbench-${jobname}" | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:" | tee -a "results/${jobname}.txt"
}

mkdir -p results
kubectl apply -k base && kubectl wait --for=condition=complete job/sysbench --timeout=60s && \
run_sysbench prepare && \
run_sysbench warmup && \
run_sysbench oltp-point-select && \
run_sysbench oltp-update-index  && \
run_sysbench oltp-update-non-index&& \
run_sysbench oltp-read-only && \
run_sysbench oltp-insert
sysbench_summary oltp-point-select && \
sysbench_summary oltp-update-index  && \
sysbench_summary oltp-update-non-index&& \
sysbench_summary oltp-read-only && \
sysbench_summary oltp-insert

# kubectl apply -k base && kubectl wait --for=condition=complete job/sysbench --timeout=60s && \
# kubectl apply -k prepare && kubectl wait --for=condition=complete job/sysbench-prepare --timeout=-1s && \
# kubectl apply -k warmup && kubectl wait --for=condition=complete job/sysbench-warmup --timeout=1200s && \
# kubectl apply -k oltp_point_select && kubectl wait --for=condition=complete job/sysbench-oltp-point-select --timeout=1200s && \
# kubectl apply -k oltp_update_index && kubectl wait --for=condition=complete job/sysbench-oltp-update-index --timeout=1200s && \
# kubectl apply -k oltp_update_non_index && kubectl wait --for=condition=complete job/sysbench-oltp-update-non-index --timeout=1200s && \
# kubectl apply -k oltp_read_only && kubectl wait --for=condition=complete job/sysbench-oltp-read-only --timeout=1200s && \
# kubectl apply -k oltp_insert && kubectl wait --for=condition=complete job/sysbench-oltp-insert --timeout=1200s && \
# kubectl logs job/sysbench-oltp-point-select | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:" && \
# kubectl logs job/sysbench-oltp-update-index  | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:" && \
# kubectl logs job/sysbench-oltp-update-non-index  | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:" && \
# kubectl logs job/sysbench-oltp-read-only | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:" && \
# kubectl logs job/sysbench-oltp-insert | egrep -e "transactions:|queries:|min:|avg:|max:|95th percentile:"
```

## get sysbench results

```shell
sysbench_summary oltp-point-select && \
sysbench_summary oltp-update-index  && \
sysbench_summary oltp-update-non-index&& \
sysbench_summary oltp-read-only && \
sysbench_summary oltp-insert
```

## delete sysbench job

```shell
kubectl delete -k oltp_insert
kubectl delete -k oltp_read_only
kubectl delete -k oltp_update_non_index
kubectl delete -k oltp_update_index
kubectl delete -k oltp_point_select
kubectl delete -k warmup
kubectl delete -k prepare
kubectl delete -k base
```
