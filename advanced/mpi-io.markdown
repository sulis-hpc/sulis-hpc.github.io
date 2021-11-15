---
layout: page
title: MPI-IO
parent: Advanced Topics
nav_order: 6
---

MPI-IO codes can show decreased performance on Sulis using the default OpenMPI IO subsystem. Performance is improved by either switching to a different File Collective algorithm in ompio or switching to using ROMIO for output. This is easily done by placing one of the following export lines in you submission script

```bash
export OMPI_MCA_io=^ompio
export OMPI_MCA_fcoll=two_phase
```

Do not put both export lines in your script, choose one or the other. You should test for which gives the better performance for your problem.
