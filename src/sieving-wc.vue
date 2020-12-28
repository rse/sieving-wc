<!--
**  Sieving-WC ~ Sieving Web Component
**  Copyright (c) 2020 Dr. Ralf S. Engelschall <rse@engelschall.com>
**
**  Permission is hereby granted, free of charge, to any person obtaining
**  a copy of this software and associated documentation files (the
**  "Software"), to deal in the Software without restriction, including
**  without limitation the rights to use, copy, modify, merge, publish,
**  distribute, sublicense, and/or sell copies of the Software, and to
**  permit persons to whom the Software is furnished to do so, subject to
**  the following conditions:
**
**  The above copyright notice and this permission notice shall be included
**  in all copies or substantial portions of the Software.
**
**  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
**  EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
**  MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
**  IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
**  CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
**  TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
**  SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
-->

<!--  HTML/Vue structure  -->
<template>
    <!--  outer frame  -->
    <div class="frame" ref="frame" part="frame">

        <!--  input area  -->
        <div ref="input" part="input" class="input">

            <!--  lower highlighting canvas (visible)  -->
            <div ref="canvas"
                class="canvas"
                v-bind:class="{ focused: focused, completing: completing }">
                <span
                    v-for="char in chars"
                    v-bind:part="'char char-' + char.type"
                    v-bind:class="{
                       char: true,
                       [ 'char-' + char.type ]: true,
                      'char-first': char.first,
                      'char-last': char.last
                    }"
                    v-html="char.char">
                </span>
            </div>

            <!--  upper editing overlay (transparent)  -->
            <textarea part="overlay"
                ref="overlay"
                class="overlay"
                v-bind:class="{ completing: completing }"
                v-on:input="overlayChange"
                v-on:scroll="overlayScroll"
                v-on:keydown="overlayKeyDown"
                v-on:keyup="overlayKeyUp"
                v-on:focus="focused = true"
                v-on:blur="focused = false"
                v-model="query$">
            </textarea>
        </div>

        <!--  auto-completion area  -->
        <div
            part="complete"
            ref="complete"
            class="complete"
            v-show="completing">
            <div v-for="(item, i) of completed"
                v-bind:ref="'item-' + i"
                class="item"
                v-bind:class="{ selected: i === completedSelected }"
                v-on:click="completedSelected = i, takeCompletion()">
                {{ item }}
            </div>
        </div>

        <!--  configuration slot  -->
        <div ref="config">
            <slot></slot>
        </div>
    </div>
</template>

<!--  CSS/LESS styling  -->
<style lang="less">
    .frame {
        width:  auto;
        height: auto;
        .input {
            width:    auto;
            height:   auto;
            position: relative;
            display:  block;
            .canvas {
                position:         absolute;
                left:             0;
                top:              0;
                width:            auto;
                height:           auto;
                font-family:      var(--font-family, monospace);
                font-size:        100%;
                padding:          4px 4px 4px 4px;
                line-break:       normal;
                line-height:      var(--line-height, 120%);
                overflow-y:       scroll;
                border:           1px solid transparent;
                border-radius:    4px;
                &.completing {
                    border-bottom-left-radius:  0;
                    border-bottom-right-radius: 0;
                }
                background-color: var(--color-bg-area, inherit);
                &.focused {
                    background-color: var(--color-bg-area-focus, inherit);
                }
                .char {
                    white-space: pre-wrap;
                }
                .char.char-first {
                    border-top-left-radius:     4px;
                    border-bottom-left-radius:  4px;
                }
                .char.char-last {
                    border-top-right-radius:    4px;
                    border-bottom-right-radius: 4px;
                }
                .char-ws {
                    color:            var(--color-fg-char-ws, inherit);
                    background-color: var(--color-bg-char-ws, inherit);
                }
                .char-union {
                    color:            var(--color-fg-char-union, inherit);
                    background-color: var(--color-bg-char-union, inherit);
                }
                .char-intersection {
                    color:            var(--color-fg-char-intersection, inherit);
                    background-color: var(--color-bg-char-intersection, inherit);
                }
                .char-subtraction {
                    color:            var(--color-fg-char-subtraction, inherit);
                    background-color: var(--color-bg-char-subtraction, inherit);
                }
                .char-group {
                    color:            var(--color-fg-char-group, inherit);
                    background-color: var(--color-bg-char-group, inherit);
                }
                .char-namespace {
                    color:            var(--color-fg-char-namespace, inherit);
                    background-color: var(--color-bg-char-namespace, inherit);
                }
                .char-regexp {
                    color:            var(--color-fg-char-regexp, inherit);
                    background-color: var(--color-bg-char-regexp, inherit);
                }
                .char-glob {
                    color:            var(--color-fg-char-glob, inherit);
                    background-color: var(--color-bg-char-glob, inherit);
                }
                .char-squoted {
                    color:            var(--color-fg-char-squoted, inherit);
                    background-color: var(--color-bg-char-squoted, inherit);
                }
                .char-dquoted {
                    color:            var(--color-fg-char-dquoted, inherit);
                    background-color: var(--color-bg-char-dquoted, inherit);
                }
                .char-bareword {
                    color:            var(--color-fg-char-bareword, inherit);
                    background-color: var(--color-bg-char-bareword, inherit);
                }
                .char-boost {
                    color:            var(--color-fg-char-boost, inherit);
                    background-color: var(--color-bg-char-boost, inherit);
                }
                .char-error {
                    color:            var(--color-fg-char-error, inherit);
                    background-color: var(--color-bg-char-error, inherit);
                }
            }
            .overlay {
                position:             absolute;
                left:                 0;
                top:                  0;
                width:                400px;
                height:               calc(3em + 10px);
                font-family:          var(--font-family, monospace);
                font-size:            100%;
                padding:              4px 4px 4px 4px;
                line-break:           normal;
                line-height:          var(--line-height, 120%);
                border:               1px solid transparent;
                background-color:     transparent;
                color:                transparent;
                border-radius:        4px;
                &.completing {
                    border-bottom-left-radius:  0;
                    border-bottom-right-radius: 0;
                }
                border-top:           1px solid var(--color-bd-area-top,    inherit);
                border-left:          1px solid var(--color-bd-area-left,   inherit);
                border-right:         1px solid var(--color-bd-area-right,  inherit);
                border-bottom:        1px solid var(--color-bd-area-bottom, inherit);
                caret-color:          var(--color-fg-caret);
                overflow-y:           scroll;
                &:focus {
                    outline:          none;
                    border-top:       1px solid var(--color-bd-area-focus-top,    inherit);
                    border-left:      1px solid var(--color-bd-area-focus-left,   inherit);
                    border-right:     1px solid var(--color-bd-area-focus-right,  inherit);
                    border-bottom:    1px solid var(--color-bd-area-focus-bottom, inherit);
                }
            }
        }
        .complete {
            position:                 absolute;
            margin-top:               3px;
            display:                  block;
            width:                    auto;
            z-index:                  1000;
            font-family:              var(--font-family, monospace);
            font-size:                100%;
            padding:                  4px 4px 4px 4px;
            background-color:         var(--color-bg-area-focus, inherit);
            color:                    var(--color-fg-completing, inherit);
            border-bottom-left-radius:   4px;
            border-bottom-right-radius:  4px;
            border-top:               0px;
            border-left:              1px solid var(--color-bd-area-focus-left,   inherit);
            border-right:             1px solid var(--color-bd-area-focus-right,  inherit);
            border-bottom:            1px solid var(--color-bd-area-focus-bottom, inherit);
            overflow-y:               scroll;
            overflow-x:               hidden;
            .item {
                width:                100%;
                cursor:               pointer;
                &:hover {
                    background-color: var(--color-bg-completing-hover, inherit);
                }
                &.selected {
                    background-color: var(--color-bg-completing-select, inherit);
                }
            }
        }
    }
</style>

<!--  JavaScript behaviour  -->
<script>
/*  import Sieving library  */
import Sieving from "sieving"

/*  helper function for exposing a Vue method as a Web Component method
    (the trick is to use a Function-based property and use the "vueComponent"
    to jump from the HTMLElement to the Vue component)  */
const exposeMethod = (name) => {
    return function (...args) {
        if (this instanceof HTMLElement && "vueComponent" in this)
            return this.vueComponent[name](...args)
        else
            return this[name](...args)
    }
}

/*  export the Vue component  */
export default {
    /*  exposed properties  */
    props: {
        query:      { type: String,   default: ""   },
        complete:   { type: String,   default: "{}" },
        blinkSpeed: { type: Number,   default: 500  },
        getQuery:   { type: Function, default: exposeMethod("getQuery$") }
    },

    /*  property change reaction (import values)  */
    watch: {
        query: function (val) {
            this.$refs.overlay.value = val
            this.overlayChange()
        },
        complete: function (val) {
            this.complete$ = JSON.parse(val)
        },
        blinkSpeed: function (val) {
            this.blinkSpeed$ = val
        }
    },

    /*  component state data  */
    data: function () {
        return {
            /*  internals of exposed properties  */
            query$:            null,
            complete$:         null,
            blinkSpeed$:       null,

            /*  pure internals  */
            sieving:           null,
            chars:             null,
            focused:           false,
            ro:                null,
            completing:        false,
            completed:         [],
            completedSelected: 0
        }
    },

    /*  component life-cycle hook: component creation  */
    created () {
        this.query$ = this.query
        this.sieving = new Sieving({})
        this.updateCanvas()
    },

    /*  component life-cycle hook: component DOM mounting  */
    mounted () {
        /*  handle resizing  */
        this.overlayResize()
        this.ro = new ResizeObserver(this.overlayResize)
            .observe(this.$refs.overlay)

        /*  take over configuration  */
        setTimeout(() => {
            const el = this.$refs.config.querySelector("script[type='application/json']")
            if (el !== null)
                this.complete$ = JSON.parse(el.innerText)
        }, 0)
    },

    /*  component life-cycle hook: component destruction  */
    destroyed () {
        delete this.ro
    },

    /*  component methods  */
    methods: {
        /*
         *  ==== EXPOSED ====
         */

        /*  getter for the current query  */
        getQuery$: function () {
            return this.$refs.overlay.value
        },

        /*
         *  ==== INTERNAL ====
         */

        /*  update the canvas  */
        updateCanvas () {
            /*  convert query string to canvas chars  */
            this.chars = []
            this.sieving.parse(this.query$, { lts: true, ast: false })
            const tokens = this.sieving.lts.slice(0, -1)
            for (const token of tokens) {
                const chars = token.text.split("")
                for (let i = 0; i < chars.length; i++) {
                    this.chars.push({
                        char:  chars[i],
                        pos:   token.pos + i,
                        type:  token.type,
                        first: i === 0,
                        last:  i === chars.length - 1,
                    })
                }
            }

            /*  adjust the canvas char first/last information  */
            for (let i = 0; i < this.chars.length; i++) {
                if (i < this.chars.length - 1 &&
                    this.chars[i].type === "namespace" &&
                    this.chars[i + 1].type !== "namespace") {
                    delete this.chars[i].last
                    delete this.chars[i + 1].first
                }
                if (i > 0 &&
                    this.chars[i - 1].type !== "boost" &&
                    this.chars[i].type === "boost") {
                    delete this.chars[i - 1].last
                    delete this.chars[i].first
                }
            }
        },

        /*  react on content change in the overlay  */
        overlayChange (ev) {
            // this.query$ = this.$refs.overlay.value
            this.updateCanvas()
            this.$emit("change", { query: this.query$ })
        },

        /*  react on scrolling in the overlay  */
        overlayScroll (ev) {
            /*  synchronize canvas  */
            this.$refs.canvas.scrollTop = this.$refs.overlay.scrollTop
        },

        /*  react on resizing in the overlay  */
        overlayResize (ev) {
            /*  determine the new size  */
            const style = window.getComputedStyle(this.$refs.overlay)
            let width  = this.$refs.overlay.clientWidth
            let height = this.$refs.overlay.clientHeight

            /*  synchronize the outer input and frame areas  */
            this.$refs.input.style.width  = width  + "px"
            this.$refs.input.style.height = height + "px"
            this.$refs.frame.style.width  = width  + "px"

            /*  calculate the inner width  */
            width  -= parseInt(style.getPropertyValue("padding-left")  .replace(/px$/, ""))
            width  -= parseInt(style.getPropertyValue("padding-right") .replace(/px$/, ""))
            height -= parseInt(style.getPropertyValue("padding-top")   .replace(/px$/, ""))
            height -= parseInt(style.getPropertyValue("padding-bottom").replace(/px$/, ""))

            /*  synchronize the inner canvas and complete areas  */
            this.$refs.canvas.style.width    = width  + "px"
            this.$refs.canvas.style.height   = height + "px"
            this.$refs.complete.style.width  = width  + "px"
            this.$refs.complete.style.height = height + "px"
        },

        /*  react on key down event in overlay  */
        overlayKeyDown (ev) {
            if (ev.code === "Space" && ev.ctrlKey === true) {
                /*  open/close completion area  */
                if (this.complete$ !== null) {
                    ev.preventDefault()
                    ev.stopPropagation()
                    if (!this.completing) {
                        this.updateComplete()
                        this.completing = true
                        this.$emit("completing", this.completing)
                    }
                    else
                        this.completing = false
                }
            }
            else if (this.completing) {
                /*  catch some keystrokes if completing...  */
                if (ev.code === "Escape") {
                    /*  close completion area  */
                    this.completing = false
                    ev.preventDefault()
                    ev.stopPropagation()
                }
                else if (ev.code === "ArrowUp") {
                    /*  select completion item (upward)  */
                    if (this.completedSelected > 0)
                        this.completedSelected--
                    ev.preventDefault()
                    ev.stopPropagation()
                    const parent = this.$refs.complete
                    const item = this.$refs[`item-${this.completedSelected}`][0]
                    const style = window.getComputedStyle(item)
                    if (item.offsetTop < parent.scrollTop)
                        parent.scrollTop = item.offsetTop
                }
                else if (ev.code === "ArrowDown") {
                    /*  select completion item (downward)  */
                    if (this.completedSelected < this.completed.length - 1)
                        this.completedSelected++
                    ev.preventDefault()
                    ev.stopPropagation()
                    const parent = this.$refs.complete
                    const item = this.$refs[`item-${this.completedSelected}`][0]
                    const style = window.getComputedStyle(item)
                    if (item.offsetTop + item.clientHeight > parent.scrollTop + parent.clientHeight)
                        parent.scrollTop = item.offsetTop + item.clientHeight - parent.clientHeight
                }
                else if (ev.code === "Enter" || ev.code === "Tab") {
                    /*  take completion item  */
                    this.takeCompletion()

                    /*  stop processing  */
                    ev.preventDefault()
                    ev.stopPropagation()
                }
            }
        },

        /*  take currently selected completion item  */
        takeCompletion () {
            if (this.completedSelected >= 0 && this.completedSelected <= this.completed.length - 1) {
                /*  determine replacement item  */
                const replacement = this.completed[this.completedSelected]

                /*  determine current completion range in overlay  */
                const { begin, cursor, end } = this.completeRange()

                /*  determine new position in overlay  */
                const pos = this.$refs.overlay.selectionStart + replacement.length - (cursor - begin)

                /*  change query (usually resets position)  */
                let query = this.$refs.overlay.value
                query = query.substring(0, begin) + replacement + query.substring(end)
                this.query$ = query
                this.updateCanvas()

                /*  stop completing  */
                this.completing = false

                /*  reset the position again  */
                setTimeout(() => {
                    this.$refs.overlay.selectionStart = pos
                    this.$refs.overlay.selectionEnd   = pos
                    this.$refs.overlay.focus()
                }, 0)
            }
        },

        /*  react on key up event in overlay  */
        overlayKeyUp (ev) {
            if (this.completing) {
                /*  update the completion items  */
                this.updateComplete()
            }
        },

        /*   determine current completion range in overlay  */
        completeRange () {
            /*  determine query  */
            const query = this.$refs.overlay.value

            /*  determine range and cursor  */
            let begin = this.$refs.overlay.selectionStart
            const end = this.$refs.overlay.selectionEnd
            let cursor = end
            if (begin === end)
                while (begin > 0 && !query.substring(begin - 1, begin).match(/^\s+$/))
                    begin--
            return { begin, cursor, end }
        },

        /*  update completion items  */
        updateComplete () {
            /*  determine current completion range in overlay  */
            const { begin, cursor, end } = this.completeRange()

            /*  determine current completion query  */
            let query = this.$refs.overlay.value
            query = query.substring(begin, end)
            let ns = ""
            let m
            if ((m = query.match(/^(?:([^:]+):|([$#%@&]))(.*)$/)) !== null) {
                ns    = m[1] || m[2]
                query = m[3]
            }

            /*  determine completed items  */
            const completed = []
            if (this.complete$[ns] !== undefined) {
                for (let item of this.complete$[ns]) {
                    if (item.length >= query.length && item.substring(0, query.length) === query) {
                        if (ns !== "")
                            item = (ns.match(/^[$#%@&]$/) ? ns : `${ns}:`) + item
                        completed.push(item)
                    }
                }
            }
            this.completed = completed

            /*  ensure the selection is still valid (again)  */
            if (this.completedSelected < 0)
               this.completedSelected = 0
            if (this.completedSelected >= this.completed.length)
               this.completedSelected = this.completed.length - 1
        }
    }
}
</script>

