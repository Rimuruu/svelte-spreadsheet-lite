<script>
  import { onMount, afterUpdate } from "svelte";
  export let header;
  export let data;
  let selection = {
    c: 0,
    r: 0,
    sc: 0,
    sr: 0
  };
  let editingText = "";
  let editing = false;
  let rowHeight = 24;
  let selectionMode = false;
  let selectionModeColumn = false;
  let headerResizeAt = -1;
  let input;
  let el;

  $: selectionCount = () => {
    return {
      r: selection.r <= selection.sr ? selection.r : selection.sr,
      c: selection.c <= selection.sc ? selection.c : selection.sc,
      w: Math.abs(selection.sc - selection.c) + 1,
      h: Math.abs(selection.sr - selection.r) + 1
    };
  };

  $: selectionSize = () => {
    return {
      r: positionLeft(selectionCount().r),
      c: positionLeft(selectionCount().c),
      w:
        positionLeft(selectionCount().c + selectionCount().w) -
        positionLeft(selectionCount().c),
      h:
        positionLeft(selectionCount().c + selectionCount().h) -
        positionLeft(selectionCount().c)
    };
  };

  $: selectionTransform = () => {
    return `translate(${positionLeft(selectionCount().c)},  ${selectionCount()
      .r * 24})`;
  };

  $: headerObj = () => {
    return header
      ? header
      : data[0].map((item, idx) => {
          return {
            name: String.fromCharCode(65 + idx),
            width: 80
          };
        });
  };

  $: positionLeft = index => {
    return headerObj()
      .slice(0, index)
      .map(it => it.width)
      .reduce((a, b) => a + b, 0);
  };

  $: editorStyleObj = () => {
    return (
      " left: " +
      positionLeft(selection.c) +
      "px;top: " +
      selection.r * 24 +
      "px;width: " +
      selectionSize().w +
      "px"
    );
  };

  function startColumnSelect(c) {
    selection.sr = data.length - 1;
    selection.r = 0;
    selection.sc = c;
    selection.c = c;
    selectionModeColumn = true;
  }

  function changeColumnSelect(c) {
    if (selectionModeColumn) {
      selection.c = c;
    }
  }
  function endColumnSelect() {
    selectionModeColumn = false;
  }
  function headerResizeStart(c) {
    headerResizeAt = c;
  }
  function headerResizeEnd() {
    headerResizeAt = -1;
  }
  let headerResizeMove = e => {
    e.preventDefault();
    var headerRect = e.target.parentNode.parentNode.getBoundingClientRect();
    var headerMouseX = e.clientX - headerRect.left;
    if (headerResizeAt >= 0) {
      const updateWidth = headerMouseX - positionLeft(headerResizeAt);
      header[headerResizeAt].width = updateWidth > 30 ? updateWidth : 30;
    }
  };
  $: widthAt = index => {
    return header[index].width;
  };

  function setDataAt(c, r, value) {
    data[r][c] = value;
  }
  function getDataAt(c, r) {
    return data[r][c];
  }
  function onBlur() {
    editing = false;
    setDataAt(selection.c, selection.r, editingText);
  }
  $: translateCol = ci => {
    return `translate(${positionLeft(ci)}, 0)`;
  };
  $: translateRow = ri => {
    return `translate(0, ${ri * 24})`;
  };
  function onMouseDownCell(c, r) {
    if (editing) {
      onBlur();
    }
    if (
      selectionCount().c === c &&
      selectionCount().r === r &&
      selectionCount().w === 1 &&
      selectionCount().h === 1
    ) {
      editHere();
      return;
    }
    setSelectionStart(c, r);
    input.focus();
  }
  function setSelectionSingle(c, r) {
    selection.c = c;
    selection.r = r;
    selection.sc = c;
    selection.sr = r;
    setEditingText();
  }
  function setSelectionStart(c, r) {
    setSelectionSingle(c, r);
    selectionMode = true;
  }
  function onMouseMoveCell(c, r) {
    if (selectionMode) {
      setSelectionEnd(c, r);
    }
  }
  let onMouseUpSvg = () => {
    endSelection();
    headerResizeEnd();
  };
  function setSelectionEnd(c, r) {
    if (selectionMode) {
      selection.c = c;
      selection.r = r;
      setEditingText();
    }
  }
  function endSelection() {
    selectionMode = false;
  }
  function editCell(c, r) {
    editing = true;
    input.focus();
  }
  function editHere() {
    editCell(selection.c, selection.r);
  }
  function clearCell(c, r) {
    setDataAt(c, r, "");
  }
  function clearSelection() {
    for (let i = 0; i < selectionCount().h; i++) {
      for (let j = 0; j < selectionCount().w; j++) {
        clearCell(selectionCount().c + j, selectionCount().r + i);
      }
    }
  }
  function isInsideTable(c, r) {
    if (c < 0) {
      return false;
    }
    if (r < 0) {
      return false;
    }
    if (c > data[0].length - 1) {
      return false;
    }
    if (r > data.length - 1) {
      return false;
    }
    return true;
  }
  function moveCursor(dc, dr) {
    if (!isInsideTable(selection.c + dc, selection.r + dr)) {
      return;
    }
    if (selectionMode) {
      setSelectionEnd(selection.c + dc, selection.r + dr);
      fixScroll();
      return;
    }
    if (editing) {
      onBlur();
    }
    setSelectionSingle(selection.c + dc, selection.r + dr);
    fixScroll();
  }
  function moveInputCaretToEnd() {
    //var el = $refs["hiddenInput"];
    input.setSelectionRange(editingText.length, editingText.length);
  }
  function fixScroll() {
    //const el = $refs["wrapper"];
    if (el.scrollTop > selection.r * 24) {
      el.scrollTop = selection.r * 24;
    }
    if (el.scrollTop < selection.r * 24 - el.clientHeight + 24) {
      el.scrollTop = selection.r * 24 - el.clientHeight + 24;
    }
    if (el.scrollLeft < positionLeft(selection.c) - el.clientWidth) {
      el.scrollLeft = positionLeft(selection.c);
    }
  }
  function setEditingText() {
    editingText = getDataAt(selection.c, selection.r);
  }

  onMount(() => {
    selection = {
      c: 0,
      r: 0,
      sc: 0,
      sr: 0
    };
    editingText = getDataAt(0, 0);

    onBlur();
  });

  function handleKeydown(e) {
    switch (e.keyCode) {
      case 8: //backspace
        if (!editing) {
          moveInputCaretToEnd();
          editHere();
        }
        break;
      case 37: //left
        moveCursor(-1, 0);
        e.preventDefault();
        break;
      case 38: //up
        moveCursor(0, -1);
        e.preventDefault();
        break;
      case 39: //right
        moveCursor(1, 0);
        e.preventDefault();
        break;
      case 40: //down
        moveCursor(0, 1);
        e.preventDefault();
        break;
      case 46: //delete
        clearSelection();
        break;
      case 13: //enter
        moveCursor(0, 1);
        break;
      case 16: //shift
        setSelectionStart(selection.c, selection.r);
        break;
      case 91: //ctrl
        break;
      case 113: //F2
        if (!editing) {
          moveInputCaretToEnd();
          editHere();
        }
        break;
      default:
        if (!editing) {
          editingText = "";
          editHere();
        }
        break;
    }
  }
  function handleKeyUp(e) {
    switch (e.keyCode) {
      case 16: //shift
        endSelection();
        break;
    }
  }
</script>

<style>
  rect {
    fill: white;
    stroke: #999;
    shape-rendering: crispEdges;
  }
  .selection {
    fill: none;
    stroke: rgb(158, 55, 255);
    stroke-width: 3px;
  }
  .col-header {
    fill: #eee;
  }
  .col-header__resize {
    cursor: col-resize;
    opacity: 0;
  }
  .col-header__resize:hover {
    cursor: col-resize;
    fill: rgb(158, 55, 255);
    opacity: 0.5;
  }
  .col-header__resize.active {
    fill: rgb(158, 55, 255);
    opacity: 0.5;
  }
  .grid {
    position: relative;
    background: #eee;
    width: 100%;
  }
  .editor__frame {
    position: absolute;
    overflow: hidden;
  }
  text {
    dominant-baseline: central;
    user-select: none;
  }
  input {
    border: none;
    box-sizing: border-box;
    outline: 0;

    padding: 0;
  }
  svg {
    display: block;
  }
  .editor__textarea {
    opacity: 0;
    width: 100%;
  }
  .editor--visible {
    opacity: 1;
  }
</style>

<svelte:body on:keydown={handleKeydown} on:keyup={handleKeyUp} />
<template>
  <div
    on:mouseup={onMouseUpSvg}
    on:mousemove={headerResizeMove}
    class="grid"
    bind:this={el}>
    <svg width={positionLeft(data.length + 1) + 1} height="24">
      {#each header as col, ci}
        <g
          key={ci}
          transform={translateCol(ci)}
          on:mousedown={() => {
            startColumnSelect(ci);
          }}
          on:mousemove={() => {
            changeColumnSelect(ci);
          }}
          on:mouseup={() => {
            endColumnSelect;
          }}>
          <rect
            class="col-header"
            x="0"
            y="0"
            width={col.width}
            height={rowHeight} />
          <text
            class="col-header__text"
            text-anchor="middle"
            x={col.width / 2}
            y="12"
            width={col.width}
            height={rowHeight}>
            {col.name}
          </text>
          <rect
            class={'col-header__resize' + (ci === headerResizeAt ? 'active' : '')}
            x={col.width - 5}
            y="0"
            width="5"
            height={rowHeight}
            on:mousedown={() => {
              headerResizeStart(ci);
            }} />
        </g>
      {/each}
    </svg>
    <div
      ref="wrapper"
      style="height: 400px; overflow: scroll; position:relative;">
      <svg width={positionLeft(data.length + 1) + 1} height={data.length * 24}>
        {#each data as row, ri}
          <g key={ri} transform={translateRow(ri)}>
            {#each row as col, ci}
              <g
                key="ci"
                transform={translateCol(ci)}
                on:mousedown={() => {
                  onMouseDownCell(ci, ri);
                }}>
                <rect x="0" y="0" height={rowHeight} width={widthAt(ci)} />
                <text x="2" y="12" height={rowHeight} width={widthAt(ci)}>
                  {col}
                </text>
              </g>
            {/each}
          </g>
        {/each}
        <rect
          transform={selectionTransform()}
          width={selectionSize().w}
          height={selectionCount().h * rowHeight}
          class="selection"
          x="0"
          y="0" />
      </svg>
      <div class="editor__frame" style={editorStyleObj()}>
        <input
          ref="hiddenInput"
          bind:this={input}
          on:mousedown={() => {
            onMouseDownCell(selection.c, selection.r);
          }}
          class={'editor__textarea ' + (editing ? 'editor--visible' : '')}
          bind:value={editingText}
          on:blur={() => {
            onBlur();
          }} />
      </div>
    </div>

  </div>
</template>
