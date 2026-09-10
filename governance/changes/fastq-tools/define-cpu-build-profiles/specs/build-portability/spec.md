# Build Portability Change Specification

## ADDED Requirements

### Requirement: Portable is the default CPU profile
Release and unspecified builds SHALL use `FQTOOLS_CPU_BASELINE=portable` and SHALL NOT implicitly require x86-64-v3 or host-native instructions.

#### Scenario: Configure default Release
- **WHEN** a user configures a Release build without a CPU profile
- **THEN** the selected profile SHALL be `portable`
- **AND** final compile commands SHALL NOT contain `-march=x86-64-v3` or `-march=native`

### Requirement: Optimized profiles are explicit
The build SHALL offer explicit `x86-64-v3` and `native` profiles without labeling either as portable.

#### Scenario: Select x86-64-v3
- **GIVEN** a supported x86-64 compiler and target
- **WHEN** the user selects `x86-64-v3`
- **THEN** final compile commands SHALL contain the intended v3 target flag

#### Scenario: Select native
- **WHEN** the user selects `native`
- **THEN** final compile commands SHALL use native tuning
- **AND** documentation SHALL state that such artifacts are not publishable general binaries

### Requirement: Invalid or unsupported profiles fail clearly
The configuration SHALL reject unknown profiles and platform-incompatible v3 selection before compilation.

#### Scenario: Unknown profile
- **WHEN** a user configures `FQTOOLS_CPU_BASELINE=typo`
- **THEN** configuration SHALL fail with the allowed values

### Requirement: Build identity exposes CPU profile
Build output or package metadata SHALL expose the selected CPU profile so artifacts are not confused.

#### Scenario: Inspect configuration summary
- **WHEN** configuration completes
- **THEN** the summary SHALL report `portable`, `x86-64-v3`, or `native`

### Requirement: Legacy native option has one migration path
The existing `ENABLE_NATIVE_ARCH` option SHALL map to the new profile only when `FQTOOLS_CPU_BASELINE` was not explicitly selected, and conflicting explicit values SHALL fail configuration.

#### Scenario: Legacy option enabled alone
- **WHEN** a user enables `ENABLE_NATIVE_ARCH` without explicitly setting the new profile
- **THEN** configuration SHALL select `native`
- **AND** SHALL print a deprecation warning with the replacement setting

#### Scenario: Legacy option conflicts with explicit portable
- **WHEN** a user enables `ENABLE_NATIVE_ARCH` and explicitly selects `portable`
- **THEN** configuration SHALL fail instead of silently choosing one

### Requirement: Runtime dispatch is not implied
Documentation SHALL NOT claim runtime SIMD dispatch or automatic fallback when the binary was compiled for a fixed optimized profile.

#### Scenario: Read portability documentation
- **WHEN** a user examines CPU compatibility
- **THEN** fixed-profile minimum CPU requirements SHALL be explicit
