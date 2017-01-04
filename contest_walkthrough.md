# Replication Walkthrough
## Introduction
To replicate a Pantheon experiment in emulation one can run `pantheon/test/run.py` inside a mahimahi container and using the `-r` flag in `pantheon/test/run.py` to connect a remote destination of `$MAHIMAHI_BASE`: `pantheon/test/run.py -r USER@IP:PANTHEON_DIR` makes ssh connections to the remote side and coordinates running each scheme.
In this case we are ssh-ing into our own machine so you will want to add your own public key to `~/.ssh/authorized_keys` and add self to `~/.ssh/known_hosts`. You should be able to run
```
mm-delay 1 sh -c 'ssh $MAHIMAHI_BASE exit'
```
Before proceeding.


## First Replication
I can generate a report for the experiment we are trying to replicate by running:
```
pantheon/analyze/analyze.py --s3-link https://stanford-pantheon.s3.amazonaws.com/real-world-results/Nepal/2017-01-03T21-30-Nepal-to-AWS-India-10-runs-logs.tar.xz
```
Looking at this, I decided to replicate using a 28 ms one way propagation delay and 10 megabits/s constant link capacity.

To perform this emulation in `pantheon/test` I first build all the schemes and install any dependencies by running:
```
pantheon/test/run.py --run-only setup
```


In performing tests under emulation I will want to use the `--run-only test` option so I do not try to build and install dependencies from inside the mahimahi shells (and ssh-ing to `$MAHIMAHI_BASE` to do it again on the same machine).


I will make a file called `10mbps_trace` which will look like this:
```
1
2
3
4
6
```
(see man mm-link for details on making an mm-link trace)


In the `pantheon/test` directory, run:
```
mm-delay 28 mm-link 10mbps_trace 10mbps_trace -- sh -c './run.py -r $USER@$MAHIMAHI_BASE:pantheon --run-only test'
```
and wait patiently to perform the experiment over the emulated link.

## Analysis
Logs of this experiment will be in `pantheon/test` directory. from here we can compare the two experiments in `pantheon/analyze` by running:
```
./compare_two_experiments.py https://stanford-pantheon.s3.amazonaws.com/real-world-results/Nepal/2017-01-03T21-30-Nepal-to-AWS-India-10-runs-logs.tar.xz ../test/
```
This returns:
```
Comparison of: 2017-01-03T21-30-Nepal-to-AWS-India-10-runs-logs and ../test/
        scheme    exp 1 runs    exp 2 runs            aggregate metric    mean 1    mean 2    % difference    std dev 1    std dev 2    % difference
--------------  ------------  ------------  --------------------------  --------  --------  --------------  -----------  -----------  --------------
     saturator            10             1         throughput (Mbit/s)      9.57      9.32          -2.65%         0.08         0.00        -100.00%
          copa            10             1         throughput (Mbit/s)      1.92      4.95        +157.83%         0.11         0.00        -100.00%
          quic            10             1         throughput (Mbit/s)      1.19      1.20          +0.97%         0.15         0.00        -100.00%
       koho_cc            10             1         throughput (Mbit/s)      8.90      9.10          +2.25%         0.14         0.00        -100.00%
        webrtc            10             1         throughput (Mbit/s)      2.86      2.28         -20.02%         0.50         0.00        -100.00%
         vegas            10             1         throughput (Mbit/s)      4.04      9.27        +129.51%         0.97         0.00        -100.00%
   default_tcp            10             1         throughput (Mbit/s)      5.08      9.30         +83.06%         1.16         0.00        -100.00%
         verus            10             1         throughput (Mbit/s)      9.24      9.25          +0.12%         0.22         0.00        -100.00%
        sprout            10             1         throughput (Mbit/s)      4.80      4.85          +1.12%         0.75         0.00        -100.00%
greg_saturator             9             1         throughput (Mbit/s)      9.34      9.12          -2.33%         0.11         0.00        -100.00%
           pcc            10             1         throughput (Mbit/s)      6.61      3.85         -41.74%         1.86         0.00        -100.00%
        scream            10             1         throughput (Mbit/s)      0.75      1.14         +51.76%         0.10         0.00        -100.00%
        ledbat            10             1         throughput (Mbit/s)      4.68      9.05         +93.29%         0.54         0.00        -100.00%
     saturator            10             1  95th percentile delay (ms)    220.21    211.45          -3.98%         5.57         0.00        -100.00%
          copa            10             1  95th percentile delay (ms)     37.25     32.97         -11.48%         0.73         0.00        -100.00%
          quic            10             1  95th percentile delay (ms)     39.27     32.85         -16.36%         0.73         0.00        -100.00%
       koho_cc            10             1  95th percentile delay (ms)     54.09     58.57          +8.29%         3.46         0.00        -100.00%
        webrtc            10             1  95th percentile delay (ms)     43.95     35.55         -19.12%         4.18         0.00        -100.00%
         vegas            10             1  95th percentile delay (ms)    126.93    146.34         +15.29%        83.59         0.00        -100.00%
   default_tcp            10             1  95th percentile delay (ms)     57.70   2019.40       +3399.89%        40.99         0.00        -100.00%
         verus            10             1  95th percentile delay (ms)    214.34    341.29         +59.23%        38.59         0.00        -100.00%
        sprout            10             1  95th percentile delay (ms)     57.12     53.71          -5.97%         0.56         0.00        -100.00%
greg_saturator             9             1  95th percentile delay (ms)    271.54    290.31          +6.91%         9.00         0.00        -100.00%
           pcc            10             1  95th percentile delay (ms)     69.96     30.87         -55.88%        29.09         0.00        -100.00%
        scream            10             1  95th percentile delay (ms)     36.34     31.14         -14.31%         0.53         0.00        -100.00%
        ledbat            10             1  95th percentile delay (ms)     43.53    128.15        +194.42%         1.79         0.00        -100.00%
     saturator            10             1                 % loss rate      1.41      0.00         -99.72%         0.30         0.00        -100.00%
          copa            10             1                 % loss rate      0.41      0.00        -100.00%         0.08         0.00        -100.00%
          quic            10             1                 % loss rate      0.45      0.00        -100.00%         0.11         0.00        -100.00%
       koho_cc            10             1                 % loss rate      0.60      0.00        -100.00%         0.13         0.00        -100.00%
        webrtc            10             1                 % loss rate      0.76      0.00        -100.00%         0.30         0.00        -100.00%
         vegas            10             1                 % loss rate      1.16      0.00        -100.00%         1.04         0.00        -100.00%
   default_tcp            10             1                 % loss rate      0.51      0.00        -100.00%         0.10         0.00        -100.00%
         verus            10             1                 % loss rate      2.69      0.00        -100.00%         1.31         0.00        -100.00%
        sprout            10             1                 % loss rate      0.66      0.00        -100.00%         0.22         0.00        -100.00%
greg_saturator             9             1                 % loss rate      3.21      0.00         -99.87%         0.89         0.00        -100.00%
           pcc            10             1                 % loss rate      1.50      0.00        -100.00%         0.86         0.00        -100.00%
        scream            10             1                 % loss rate      0.39      0.00        -100.00%         0.08         0.00        -100.00%
        ledbat            10             1                 % loss rate      0.49      0.00        -100.00%         0.11         0.00        -100.00%
```

## Next steps
This is a start, but we can do better. Seeing from the analysis output that all schemes on the Nepal trace got at least 0.4% loss I will add this to my emulation using more mahimahi shells:
```
mm-delay 28 mm-loss uplink .004 mm-loss downlink .004 mm-link 10mbps_trace 10mbps_trace -- sh -c './run.py -r $USER@$MAHIMAHI_BASE:pantheon --run-only test'
```

With this `compare_two_experiments.py` gets:
```
Comparison of: 2017-01-03T21-30-Nepal-to-AWS-India-10-runs-logs and ../test
        scheme    exp 1 runs    exp 2 runs            aggregate metric    mean 1    mean 2    % difference    std dev 1    std dev 2    % difference
--------------  ------------  ------------  --------------------------  --------  --------  --------------  -----------  -----------  --------------
     saturator            10             1         throughput (Mbit/s)      9.57      9.27          -3.12%         0.08         0.00        -100.00%
          copa            10             1         throughput (Mbit/s)      1.92      4.37        +128.00%         0.11         0.00        -100.00%
          quic            10             1         throughput (Mbit/s)      1.19      1.15          -3.06%         0.15         0.00        -100.00%
       koho_cc            10             1         throughput (Mbit/s)      8.90      9.04          +1.57%         0.14         0.00        -100.00%
        webrtc            10             1         throughput (Mbit/s)      2.86      2.93          +2.59%         0.50         0.00        -100.00%
         vegas            10             1         throughput (Mbit/s)      4.04      3.84          -4.90%         0.97         0.00        -100.00%
   default_tcp            10             1         throughput (Mbit/s)      5.08      3.98         -21.67%         1.16         0.00        -100.00%
         verus            10             1         throughput (Mbit/s)      9.24      8.40          -9.13%         0.22         0.00        -100.00%
        sprout            10             1         throughput (Mbit/s)      4.80      4.74          -1.13%         0.75         0.00        -100.00%
greg_saturator             9             1         throughput (Mbit/s)      9.34      9.07          -2.86%         0.11         0.00        -100.00%
           pcc            10             1         throughput (Mbit/s)      6.61      4.53         -31.47%         1.86         0.00        -100.00%
        scream            10             1         throughput (Mbit/s)      0.75      0.48         -36.00%         0.10         0.00        -100.00%
        ledbat            10             1         throughput (Mbit/s)      4.68      5.53         +18.07%         0.54         0.00        -100.00%
     saturator            10             1  95th percentile delay (ms)    220.21    211.55          -3.94%         5.57         0.00        -100.00%
          copa            10             1  95th percentile delay (ms)     37.25     33.15         -11.00%         0.73         0.00        -100.00%
          quic            10             1  95th percentile delay (ms)     39.27     37.27          -5.11%         0.73         0.00        -100.00%
       koho_cc            10             1  95th percentile delay (ms)     54.09     58.03          +7.30%         3.46         0.00        -100.00%
        webrtc            10             1  95th percentile delay (ms)     43.95     38.13         -13.26%         4.18         0.00        -100.00%
         vegas            10             1  95th percentile delay (ms)    126.93     34.37         -72.92%        83.59         0.00        -100.00%
   default_tcp            10             1  95th percentile delay (ms)     57.70     48.28         -16.33%        40.99         0.00        -100.00%
         verus            10             1  95th percentile delay (ms)    214.34     67.56         -68.48%        38.59         0.00        -100.00%
        sprout            10             1  95th percentile delay (ms)     57.12     54.14          -5.22%         0.56         0.00        -100.00%
greg_saturator             9             1  95th percentile delay (ms)    271.54    291.40          +7.31%         9.00         0.00        -100.00%
           pcc            10             1  95th percentile delay (ms)     69.96     30.86         -55.89%        29.09         0.00        -100.00%
        scream            10             1  95th percentile delay (ms)     36.34     31.49         -13.36%         0.53         0.00        -100.00%
        ledbat            10             1  95th percentile delay (ms)     43.53     37.35         -14.18%         1.79         0.00        -100.00%
     saturator            10             1                 % loss rate      1.41      0.43         -69.45%         0.30         0.00        -100.00%
          copa            10             1                 % loss rate      0.41      0.44          +6.62%         0.08         0.00        -100.00%
          quic            10             1                 % loss rate      0.45      0.54         +18.80%         0.11         0.00        -100.00%
       koho_cc            10             1                 % loss rate      0.60      0.36         -39.90%         0.13         0.00        -100.00%
        webrtc            10             1                 % loss rate      0.76      0.40         -47.24%         0.30         0.00        -100.00%
         vegas            10             1                 % loss rate      1.16      0.37         -68.21%         1.04         0.00        -100.00%
   default_tcp            10             1                 % loss rate      0.51      0.36         -29.83%         0.10         0.00        -100.00%
         verus            10             1                 % loss rate      2.69      0.44         -83.58%         1.31         0.00        -100.00%
        sprout            10             1                 % loss rate      0.66      0.39         -41.48%         0.22         0.00        -100.00%
greg_saturator             9             1                 % loss rate      3.21      0.48         -85.07%         0.89         0.00        -100.00%
           pcc            10             1                 % loss rate      1.50      0.45         -69.95%         0.86         0.00        -100.00%
        scream            10             1                 % loss rate      0.39      0.59         +53.86%         0.08         0.00        -100.00%
        ledbat            10             1                 % loss rate      0.49      0.38         -22.86%         0.11         0.00        -100.00%
```

This actually worked even better than I thought it would! Multiple schemes are withing single digit % differences for throughput and 95th percentile delay.


To try to get more loss for high throughput schemes I will add some queueing loss in mm-link. Lets try a 100 packet droptail queue on top of what we have already:
```
mm-delay 28 mm-loss uplink .004 mm-loss downlink .004 mm-link 10mbps_trace 10mbps_trace --uplink-queue="droptail" --uplink-queue-args="packets=50" -- sh -c './run.py -r $USER@$MAHIMAHI_BASE:pantheon --run-only test'
```

This gets:
```
Comparison of: 2017-01-03T21-30-Nepal-to-AWS-India-10-runs-logs and ../test/
        scheme    exp 1 runs    exp 2 runs            aggregate metric    mean 1    mean 2    % difference    std dev 1    std dev 2    % difference
--------------  ------------  ------------  --------------------------  --------  --------  --------------  -----------  -----------  --------------
     saturator            10             1         throughput (Mbit/s)      9.57      9.27          -3.09%         0.08         0.00        -100.00%
          copa            10             1         throughput (Mbit/s)      1.92      4.69        +144.56%         0.11         0.00        -100.00%
          quic            10             1         throughput (Mbit/s)      1.19      1.18          -0.95%         0.15         0.00        -100.00%
       koho_cc            10             1         throughput (Mbit/s)      8.90      9.05          +1.66%         0.14         0.00        -100.00%
        webrtc            10             1         throughput (Mbit/s)      2.86      2.85          -0.35%         0.50         0.00        -100.00%
         vegas            10             1         throughput (Mbit/s)      4.04      3.54         -12.29%         0.97         0.00        -100.00%
   default_tcp            10             1         throughput (Mbit/s)      5.08      4.01         -21.02%         1.16         0.00        -100.00%
         verus            10             1         throughput (Mbit/s)      9.24      8.23         -10.94%         0.22         0.00        -100.00%
        sprout            10             1         throughput (Mbit/s)      4.80      4.75          -1.02%         0.75         0.00        -100.00%
greg_saturator             9             1         throughput (Mbit/s)      9.34      9.06          -3.02%         0.11         0.00        -100.00%
           pcc            10             1         throughput (Mbit/s)      6.61      4.56         -30.99%         1.86         0.00        -100.00%
        scream            10             1         throughput (Mbit/s)      0.75      0.81          +8.38%         0.10         0.00        -100.00%
        ledbat            10             1         throughput (Mbit/s)      4.68      5.18         +10.60%         0.54         0.00        -100.00%
     saturator            10             1  95th percentile delay (ms)    220.21    150.31         -31.74%         5.57         0.00        -100.00%
          copa            10             1  95th percentile delay (ms)     37.25     33.37         -10.42%         0.73         0.00        -100.00%
          quic            10             1  95th percentile delay (ms)     39.27     31.93         -18.71%         0.73         0.00        -100.00%
       koho_cc            10             1  95th percentile delay (ms)     54.09     58.35          +7.88%         3.46         0.00        -100.00%
        webrtc            10             1  95th percentile delay (ms)     43.95     39.30         -10.58%         4.18         0.00        -100.00%
         vegas            10             1  95th percentile delay (ms)    126.93     57.57         -54.65%        83.59         0.00        -100.00%
   default_tcp            10             1  95th percentile delay (ms)     57.70    120.94        +109.60%        40.99         0.00        -100.00%
         verus            10             1  95th percentile delay (ms)    214.34     89.44         -58.28%        38.59         0.00        -100.00%
        sprout            10             1  95th percentile delay (ms)     57.12     53.14          -6.96%         0.56         0.00        -100.00%
greg_saturator             9             1  95th percentile delay (ms)    271.54    150.00         -44.76%         9.00         0.00        -100.00%
           pcc            10             1  95th percentile delay (ms)     69.96     31.07         -55.59%        29.09         0.00        -100.00%
        scream            10             1  95th percentile delay (ms)     36.34     31.18         -14.21%         0.53         0.00        -100.00%
        ledbat            10             1  95th percentile delay (ms)     43.53     36.66         -15.78%         1.79         0.00        -100.00%
     saturator            10             1                 % loss rate      1.41     23.55       +1571.34%         0.30         0.00        -100.00%
          copa            10             1                 % loss rate      0.41      0.37         -10.53%         0.08         0.00        -100.00%
          quic            10             1                 % loss rate      0.45      0.18         -60.16%         0.11         0.00        -100.00%
       koho_cc            10             1                 % loss rate      0.60      0.40         -33.14%         0.13         0.00        -100.00%
        webrtc            10             1                 % loss rate      0.76      0.38         -49.73%         0.30         0.00        -100.00%
         vegas            10             1                 % loss rate      1.16      0.52         -55.47%         1.04         0.00        -100.00%
   default_tcp            10             1                 % loss rate      0.51      0.67         +31.43%         0.10         0.00        -100.00%
         verus            10             1                 % loss rate      2.69      2.11         -21.42%         1.31         0.00        -100.00%
        sprout            10             1                 % loss rate      0.66      0.39         -41.12%         0.22         0.00        -100.00%
greg_saturator             9             1                 % loss rate      3.21     10.67        +232.36%         0.89         0.00        -100.00%
           pcc            10             1                 % loss rate      1.50      0.49         -67.42%         0.86         0.00        -100.00%
        scream            10             1                 % loss rate      0.39      0.28         -27.72%         0.08         0.00        -100.00%
        ledbat            10             1                 % loss rate      0.49      0.42         -14.86%         0.11         0.00        -100.00%
```

This looks a little worse than our previous attempt. I will have to go back to the drawing board to improve from here.

## Note
This example only did one run for each scheme in an experiment. To start looking at the mean and standard deviation over multiple runs of emulation add `--run-times n` to `run.py`.

