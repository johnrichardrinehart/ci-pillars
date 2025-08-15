Continuous integration (CI) systems should be supported by certain general
design considerations (or guiding principles) in order that they maintain their
value as the use of the underlying integration system increases - both in terms
of the number of developers and the complexity of the build systems and
software they execute in CI. Below we outline these considerations at a
high level, accompanied by lower-level sub-items, in order to inform any
technical implementation. This list is unordered so as to emphasize equal
consideration for each of these general principles (of course, though, any
implementation should define specific technical goals based on these principles
and triage those technical goals according to the organization’s current
needs/deficiencies).

* Reliable
   1. Failures should not be resolved by retrying;
   2. Different CI jobs running in the same compute environment should not
      affect the outcome of another job;
   3. Failures are only related to relevant code changes (not invalid auth
      credentials, or disk space issues, or time of day, etc.);
* Available
   1. Jobs don’t wait unreasonable amounts of time in order to begin executing;
* Efficient
   1. Jobs utilize a set of system resources (disk IOPS, disk space, CPU
      cycles, RSS allocations, network I/O, etc.) commensurate with the job’s
      requirements;
   2. Inefficient jobs are right-sized (ideally via automation but at least via
      human intervention, supported by automatically-captured historical
      metrics);
   3. CI execution environments shouldn't survive longer than is cost-effective
      (note: long provisioning times for some execution environments may tip
      the balance in favor toward quasi-ephemeral vs. purely ephemeral);
* Secure
   1. The permissions of users in CI should be scoped to the minimum set
      required for the job (not root by default);
   2. Credentials required for a job to execute successfully should be
      short-lived (preferably provided by a 3rd-party auth service and
      provisioned per-job for auditing purposes) and not written to the filesystem;
   3. Network configuration of the CI execution environment should not permit
      unnecessary egress/ingress;
   4. CI artifacts which are intended to be consumed by production devices
      should be signed by a trusted authority and associated/registered with
      the CI job which produced it;
* Clear
   1. Execution environment version and definition should be easy to
      reproduce/introspect if only the CI job output is available (i.e. the job
      indicates the OCI image, tag, and hash used for the job or the job indicates
      the NixOS toplevel path used for the job);
   2. Log lines should be limited to reporting information which helps diagnose
      a failure (lines indicating success should be discouraged), artifacts
      produced as a side-effect of a job should only provide useful artifacts in
      successful cases, etc.;
   3. In the case of failure, CI jobs should provide a single reproduction
      command which developers can use to execute all steps within the job
      within the same execution environment outside of CI, if desired;
* Supportive
   1. CI should work for the developer, not against them. Unavoidably
      long-running or flaky jobs which do not serve the needs of
      high-velocity repositories/branches should be pruned (flagged automatically via
      telemetry);
   2. In the case of failure, a CI system should egress as much work to a cache
      as possible in order to avoid a developer’s machine doing a lot of the
      same work in order to reproduce;
* Performant
   1. Inter-job dependencies should not unnecessarily block progress (fan-ins
      should be used sparingly);
   2. Jobs should not define steps which unnecessarily block progress (e.g. if
      2 things can be fetched in parallel, but are fetched sequentially, then
      rewrite in order to fetch simultaneously)
* Observable
   1. Relevant monitoring/telemetry (backend) services run at different layers
      of the stack to provide insight on resource bottlenecks;
   2. CI execution environments should automatically report host metrics
      capturing things like:
      1. version of the CI execution environment (OCI image name+tag+hash,
         NixOS /run/current-system, etc.)
      2. `machine-id`
      3. available memory
      4. Disk I/O metrics (available space, IOPS, etc.)
      5. network ingress/egress
      6. unused CPU
   3. CI execution environments or jobs should work to instrument the PID tree
      for their jobs to emit metrics related to per-PID resource usage;
   4. CI logs should be visible within the CI UI but also centrally ingested
      (helps with cases like finding all jobs which failed in some specific,
      identifiable way);
* Flexible
   1. CI systems should easily provide support for execution environments which
      correspond to the build platforms used by developers and host platforms
      associated with the runtime environment of the developed software (where these
      platforms may be changing in time);
   2. CI systems should provide first-class support for rolling out execution
      environment updates (newer OCI runtime version or variant, new CI AMI
      releases, etc.);
   3. CI systems should be “premises-agnostic”, wherever practical (able to run
      as easily locally as on-prem as within various cloud environments).
   4. Abstractions over common CI operations should be wrapped in type-safe
      “modules” (type-safe, here, means that exposed parameters are typed),
      consumable in a way that’s independent of the specific CI system (no
      Actions or Orbs or Components)
