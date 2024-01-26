#!/usr/bin/env python3

"""Set up Qiime 2 on Google colab.

Do not use this on o local machine, especially not as an admin!
"""

import os
import sys
import shutil
from subprocess import Popen, PIPE

r = Popen(["pip", "install", "rich"])
r.wait()
from rich.console import Console  # noqa
con = Console()

PREFIX = "/usr/local/miniforge3/"

has_conda = "conda version" in os.popen("%s/bin/conda info" % PREFIX).read()
has_qiime = "QIIME 2 release:" in os.popen("qiime info").read()


MINICONDA_PATH = (
    "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"
)

QIIME_YAML_TEMPLATE = (
    "https://data.qiime2.org/distro/amplicon/qiime2-amplicon-{version}-py{python}-linux-conda.yml"
)

if len(sys.argv) == 2:
    version = sys.argv[1]
else:
    version = "2023.9"

if tuple(float(v) for v in version.split(".")) < (2021, 4):
    pyver = "36"
else:
    pyver = "38"

CONDA = "mamba"
CONDA_ARGS = ["-q"]

if tuple(float(v) for v in version.split(".")) < (2023, 9):
    QIIME_YAML_TEMPLATE = (
        "https://data.qiime2.org/distro/core/qiime2-{version}-py{python}-linux-conda.yml"
    )

QIIME_YAML_URL = QIIME_YAML_TEMPLATE.format(version=version, python=pyver)
QIIME_YAML = os.path.basename(QIIME_YAML_URL)


def cleanup():
    """Remove downloaded files."""
    if os.path.exists(os.path.basename(MINICONDA_PATH)):
        os.remove(os.path.basename(MINICONDA_PATH))
    if os.path.exists(QIIME_YAML):
        os.remove(QIIME_YAML)
    if os.path.exists("/content/sample_data"):
        shutil.rmtree("/content/sample_data")
    con.log(":broom: Cleaned up unneeded files.")


def run_and_check(args, check, message, failure, success, console=con):
    """Run a command and check that it worked."""
    console.log(message)
    r = Popen(args, env=os.environ, stdout=PIPE, stderr=PIPE,
              universal_newlines=True)
    o, e = r.communicate()
    out = o + e
    if r.returncode == 0 and check in out:
        console.log("[blue]%s[/blue]" % success)
    else:
        console.log("[red]%s[/red]" % failure, out)
        cleanup()
        sys.exit(1)


if __name__ == "__main__":
    if not has_conda:
        run_and_check(
            ["wget", MINICONDA_PATH],
            "saved",
            ":snake: Downloading miniforge...",
            "failed downloading miniforge :sob:",
            ":snake: Done."
        )

        run_and_check(
            ["bash", os.path.basename(MINICONDA_PATH), "-bfp", PREFIX],
            "installation finished.",
            ":snake: Installing miniforge...",
            "could not install miniforge :sob:",
            ":snake: Installed miniforge to `/usr/local`."
        )
    else:
        con.log(":snake: Miniforge is already installed. Skipped.")

    if not has_qiime:
        run_and_check(
            ["wget", QIIME_YAML_URL],
            "saved",
            ":mag: Downloading Qiime 2 package list...",
            "could not download package list :sob:",
            ":mag: Done."
        )

        run_and_check(
            [PREFIX + "bin/" + CONDA, "env", "create", *CONDA_ARGS, "--prefix", "/usr/local", "--file", QIIME_YAML],
            "Verifying transaction: ...working... done",
            ":mag: Installing Qiime 2. This may take a little bit.\n :clock1:",
            "could not install Qiime 2 :sob:",
            ":mag: Done."
        )

        run_and_check(
            ["pip", "install", "empress"],
            "Successfully installed empress-",
            ":evergreen_tree: Installing Empress...",
            "could not install Empress :sob:",
            ":evergreen_tree: Done."
        )
    else:
        con.log(":mag: Qiime 2 is already installed. Skipped.")

    run_and_check(
        ["qiime", "info"],
        "QIIME 2 release:",
        ":bar_chart: Checking that Qiime 2 command line works...",
        "Qiime 2 command line does not seem to work :sob:",
        ":bar_chart: Qiime 2 command line looks good :tada:"
    )

    if sys.version_info[0:2] == (int(pyver[0]), int(pyver[1])):
        sys.path.append("/usr/local/lib/python3.{}/site-packages".format(pyver[1]))
        con.log(":mag: Fixed import paths to include Qiime 2.")

        con.log(":bar_chart: Checking if Qiime 2 import works...")
        try:
            import qiime2  # noqa
        except Exception:
            con.log("[red]Qiime 2 can not be imported :sob:[/red]")
            sys.exit(1)
        con.log("[blue]:bar_chart: Qiime 2 can be imported :tada:[/blue]")

    cleanup()

    con.log("[green]Everything is A-OK. "
            "You can start using Qiime 2 now :thumbs_up:[/green]")
