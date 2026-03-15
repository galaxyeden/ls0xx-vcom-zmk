# Copyright (c) 2026 eden ariel - eden@rainbowtree.house
# SPDX-License-Identifier: Apache-2.0

zephyr_library_amend()
zephyr_library_sources_ifdef(CONFIG_LS0XX_ZMK ls0xx_zmk.c)