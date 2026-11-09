# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
50.1
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->237.2<!--/answer--> | <!--answer:vm_warm_median_ms-->211.4<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->2261.5<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->642.0<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1497.3<!--/answer--> | <!--answer:function_warm_median_ms-->639.8<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
VM          #############                            211.4 ms
Cloud Run   ########################################  642.0 ms
Function    ########################################  639.8 ms

The VM's first request is not a true cold start, since its container was already running. For Cloud Run and Cloud Functions, the first request was measured after an idle period, although this does not guarantee that the instances had scaled to zero.
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The IaaS VM does not scale automatically. All requests go to our single e2-micro VM, so performance is limited by its resources. Cloud Run can automatically create more container instances when traffic increases, but we limited it to three with --max-instances=3. Cloud Functions Gen 2 also scales automatically by creating new instances. Since we did not set a maximum number of instances, it follows Google's default limits and quotas. In our configuration, each function instance handles one request at a time.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
For the IaaS deployment, I created a custom VPC and subnet, two firewall rules (HTTP on port 8080 and SSH on port 22), an e2-micro VM, and SSH access. Inside the VM, I installed Docker, authenticated to Artifact Registry, pulled the image and launched the container. Cloud Run was simpler: I reused the same image and specified public access, port 8080 and autoscaling settings. For Cloud Functions, I deployed the Python source with the echo entry point and an HTTP trigger rather than supplying a container image. The managed paths also required IAM troubleshooting for Cloud Build and Artifact Registry, but no VM administration.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
SERVICE: echo
REGION: us-central1
URL: https://echo-592496180948.us-central1.run.app
LAST DEPLOYED BY: service-592496180948@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-11-06T13:38:23.665288Z

SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-592496180948.us-central1.run.app
LAST DEPLOYED BY: rodrigo.de.la-nuez@eurecom.fr
LAST DEPLOYED AT: 2026-11-06T13:28:20.195449Z

<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The command shows two Cloud Run services: `echo-bot`, which I deployed using Docker, and `echo`, which was automatically created when I deployed the Cloud Function. This confirms that Cloud Functions Gen 2 uses Cloud Run infrastructure. Since both can scale to zero and start new instances when needed, their response times are quite similar. Cloud Run took 2261.5 ms for the first request, compared to 1497.3 ms for Cloud Functions, while their warm latencies were almost identical (642.0 and 639.8 ms). However, one measurement is not enough to draw general conclusions about performance. The main difference is how we deploy them: Cloud Run uses our Docker image, while Cloud Functions only requires the Python code and an entry point, leaving the rest to Google.
<!--/answer-->
