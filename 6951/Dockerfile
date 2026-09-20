FROM python:3.10.12-bookworm

ARG BASE_COMMIT=3441f68a7b9584e74e318aafd66f03edff0cf5a8

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV SETUPTOOLS_SCM_PRETEND_VERSION=1.20.0

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL "https://codeload.github.com/holoviz/holoviews/tar.gz/${BASE_COMMIT}" \
      -o /tmp/src.tgz \
 && tar tzf /tmp/src.tgz | head -1 | grep -qF "holoviews-${BASE_COMMIT}/" \
 && mkdir -p /app \
 && tar xzf /tmp/src.tgz --strip-components=1 -C /app \
 && rm /tmp/src.tgz

WORKDIR /app

RUN pip install --no-cache-dir -e ".[recommended]" \
 && pip install --no-cache-dir pytest pytest-rerunfailures pytest-asyncio coverage

# Conftest exit hook v2 (D11)
RUN printf 'import os as _jf_os, sys as _jf_sys, pytest as _jf_pytest\n\
_JF_EXIT = {"status": 0}\n\
@_jf_pytest.hookimpl(trylast=True)\n\
def pytest_sessionfinish(session, exitstatus):\n\
    _JF_EXIT["status"] = int(exitstatus)\n\
@_jf_pytest.hookimpl(trylast=True)\n\
def pytest_unconfigure(config):\n\
    _jf_sys.stdout.flush(); _jf_sys.stderr.flush()\n\
    _jf_os._exit(_JF_EXIT["status"])\n' >> /app/conftest.py

RUN python -m pytest holoviews/tests/plotting/bokeh/test_graphplot.py::TestBokehTriMeshPlot::test_trimesh_op_edge_color_linear_mean_node -v -o addopts=
