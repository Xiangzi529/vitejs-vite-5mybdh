<script setup>
import '@univerjs/design/lib/index.css';
import '@univerjs/ui/lib/index.css';
import '@univerjs/docs-ui/lib/index.css';
import '@univerjs/sheets-ui/lib/index.css';
import '@univerjs/sheets-formula/lib/index.css';

import { LocaleType, Tools, Univer, UniverInstanceType } from '@univerjs/core';
import { defaultTheme } from '@univerjs/design';

import { UniverFormulaEnginePlugin } from '@univerjs/engine-formula';
import { UniverRenderEnginePlugin } from '@univerjs/engine-render';

import { UniverUIPlugin } from '@univerjs/ui';

import { UniverDocsPlugin } from '@univerjs/docs';
import { UniverDocsUIPlugin } from '@univerjs/docs-ui';

import { UniverSheetsPlugin } from '@univerjs/sheets';
import { UniverSheetsFormulaPlugin } from '@univerjs/sheets-formula';
import { UniverSheetsUIPlugin } from '@univerjs/sheets-ui';
import { FUniver } from '@univerjs/facade';
import DesignZhCN from '@univerjs/design/locale/zh-CN';
import UIZhCN from '@univerjs/ui/locale/zh-CN';
import DocsUIZhCN from '@univerjs/docs-ui/locale/zh-CN';
import SheetsZhCN from '@univerjs/sheets/locale/zh-CN';
import SheetsUIZhCN from '@univerjs/sheets-ui/locale/zh-CN';
const univerApis = [];
const createUniverSheet = (containerId) => {
  const univer = new Univer({
    theme: defaultTheme,
    locale: LocaleType.ZH_CN,
    locales: {
      [LocaleType.ZH_CN]: Tools.deepMerge(
        SheetsZhCN,
        DocsUIZhCN,
        SheetsUIZhCN,
        UIZhCN,
        DesignZhCN
      ),
    },
  });

  univer.registerPlugin(UniverRenderEnginePlugin);
  univer.registerPlugin(UniverFormulaEnginePlugin);

  univer.registerPlugin(UniverUIPlugin, {
    container: containerId,
  });

  univer.registerPlugin(UniverDocsPlugin, {
    hasScroll: false,
  });
  univer.registerPlugin(UniverDocsUIPlugin);

  univer.registerPlugin(UniverSheetsPlugin);
  univer.registerPlugin(UniverSheetsUIPlugin);
  univer.registerPlugin(UniverSheetsFormulaPlugin);

  univer.createUnit(UniverInstanceType.UNIVER_SHEET, {});
  univerApis[containerId] = FUniver.newAPI(univer);
  const commonCmds = [
    'sheet.operation.set-scroll',
    'sheet.operation.set-selections',
    'sheet.operation.set-worksheet-active',
    'sheet.command.set-zoom-ratio',
    'sheet.mutation.set-col-visible',
    'sheet.mutation.set-col-hidden',
    'sheet.mutation.set-row-visible',
    'sheet.mutation.set-row-hidden',
  ];
  univerApis[containerId].onCommandExecuted((command) => {
    const { id, params } = command;
    if (params && params._fromListener) {
      return;
    }
    if (commonCmds.includes(id)) {
      Object.keys(univerApis).forEach((key) => {
        if (key !== containerId) {
          const newParams = { ...params, _fromListener: true };
          univerApis[key] && univerApis[key].executeCommand(id, newParams);
        }
      });
    }
  });
};
createUniverSheet('left');
createUniverSheet('right');
</script>

<template>
  <div class="container">
    <div id="left"></div>
    <div id="right"></div>
  </div>
</template>

<style scoped lang="scss">
.container {
  height: 100vh;
  background-color: black;
  display: flex;
  > div {
    flex: 1;
    height: 100%;
  }
}
</style>
