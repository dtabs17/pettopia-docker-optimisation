# Pettopia Docker Optimisation

**Reducing Docker image size and measuring build performance for a Java Spring Boot backend.**

This project investigates how Docker packaging affects the size and build time of the backend for [Pettopia Online Pet Store](https://github.com/dtabs17/Pettopia-Online-Pet-Store). It is a separate engineering project using Pettopia as the test application, not a copy of the main application repository.

The experiment compared a single-stage Docker image against a multi-stage build that uses a Java Development Kit (JDK) to compile the application and a Java Runtime Environment (JRE) to run the packaged JAR.

## Results

| Metric | Single-stage baseline | Multi-stage build | Observed change |
| --- | ---: | ---: | ---: |
| Docker image disk usage | 917 MB | 557 MB | **39.3% smaller** |
| Docker-reported content size | 309 MB | 163 MB | **47.2% smaller** |
| Mean no-cache build time (5 runs each) | 99.872 s | 68.379 s | **31.5% lower** |

The primary goal was to reduce final image size. The build-time difference is a secondary **observed result**, not a guarantee of faster builds on every machine. Both groups were tested sequentially on the same laptop, with five completed no-cache builds per configuration. Other factors such as Maven downloads, network conditions and system load were not independently controlled.

## The problem

The baseline Dockerfile used a Java 25 JDK image to compile the Spring Boot application and also retained that same environment in the finished runtime image. Image inspection showed unnecessary build-time content remaining alongside the executable JAR, including:

- The full JDK environment, rather than a runtime-only Java image.
- The local Maven repository (`/root/.m2`), approximately 115 MB.
- Source files (`/app/src`), approximately 5.5 MB.
- Build output (`/app/target`), approximately 73 MB, including the required JAR.

These are different filesystem and layer measurements, so they should not be added together to calculate image size.

## The change

The multi-stage Dockerfile separates compilation from runtime packaging:

1. **Build stage:** Uses `eclipse-temurin:25-jdk-jammy` and the Maven wrapper to package the Spring Boot application.
2. **Runtime stage:** Starts from `eclipse-temurin:25-jre-jammy` and copies only the executable JAR from the build stage.

The application source and Maven packaging command were kept the same for both configurations. The improvement combined two changes, a separate runtime stage and a JRE-based final image. Their individual contributions were not measured separately.

The optimised image was checked for a working Java executable (OpenJDK 25.0.5) and the packaged application JAR (approximately 63 MB).

## Method

The comparison followed a practical **Define, Measure, Analyse, Improve, Control (DMAIC)** structure:

- **Define:** Set image size as the primary measure and full Docker build time as a secondary measure.
- **Measure:** Recorded image-size readings, inspected Docker layers and directories, and collected five no-cache elapsed times for each Dockerfile.
- **Analyse:** Compared descriptive statistics, visualised possible causes and assessed the difference between the two sets of build times. Used a fishbone diagram, cause-and-effect matrix, Pareto chart, FMEA and statistical checks.
- **Improve:** Selected and implemented a two-stage JDK/JRE Dockerfile without changing application features.
- **Control:** Proposed checks for later image-size regressions and build-time changes. The control plan was not operated longitudinally.

### Recorded build times

| Run | Baseline (s) | Multi-stage (s) |
| ---: | ---: | ---: |
| 1 | 104.023 | 72.084 |
| 2 | 102.713 | 70.806 |
| 3 | 98.173 | 78.337 |
| 4 | 96.614 | 61.424 |
| 5 | 97.835 | 59.246 |

The pilot runs and an additional photo-evidence build were excluded from these samples. The sample averages were 99.8716 seconds for the baseline and 68.3794 seconds for the optimised version.

## Tools and technologies

**Java 25, Spring Boot, Maven Wrapper, Docker, Docker Desktop, Docker BuildKit, WSL2/Ubuntu, Bash, PowerShell, Windows 11, Excel, diagrams.net (draw.io), Python and SciPy.**

Docker and shell tools were used for building, timing and image inspection. Excel, diagrams.net, Python and SciPy supported the investigation and analysis. They were not embedded into Pettopia's application code as part of this project.

## Scope and limitations

This project evaluates **Docker image construction and packaging**, not a production deployment of the pet store. The Java runtime and JAR were verified in the optimised image. A separate database-backed startup attempt could not complete because the test environment's MySQL connection was refused. Full application deployment was outside the optimisation scope, and no claim of an end-to-end working containerised stack is made here.

The measurements were collected in **November 2026** using a particular working copy of Pettopia's backend. The [main Pettopia repository](https://github.com/dtabs17/Pettopia-Online-Pet-Store) continues to evolve, so these measurements should not be treated as benchmarks of its latest version.

The experiment used five sequential builds per configuration, not a randomised test, and did not measure long-term build consistency or CI/CD performance. Maven tests were skipped during packaging in both configurations.

## Related project

**[Pettopia Online Pet Store](https://github.com/dtabs17/Pettopia-Online-Pet-Store)** is the application whose backend was used for this experiment. Its ongoing feature development and codebase are maintained separately from this Docker optimisation project.

## Documentation and evidence

The project has a separate, detailed engineering record covering the troubleshooting steps, original Dockerfiles, exact benchmark commands, image-layer inspections, raw timings, analysis and known limitations. The Dockerfiles and supporting evidence will be added to this repository separately after review.
