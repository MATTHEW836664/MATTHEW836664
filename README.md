
function createShader(ctx, type, src) {
    const s = ctx.createShader(type);
    ctx.shaderSource(s, src);
    ctx.compileShader(s);
    if (!ctx.getShaderParameter(s, ctx.COMPILE_STATUS))
        throw "error compile " + (type === ctx.VERTEX_SHADER? "vertex": "fragment") + " shader: "
            + ctx.getShaderInfoLog(s);
    return s;
}

function createProgram(ctx, vertShader, fragShader) {
    const p = ctx.createProgram();
    ctx.attachShader(p, vertShader);
    ctx.attachShader(p, fragShader);
    ctx.linkProgram(p);
    if (!ctx.getProgramParameter(p, ctx.LINK_STATUS))
        throw "error link program: " + ctx.getProgramInfoLog(p);
    return p;
}

class Program {
    constructor(ctx, vertSrc, fragSrc) {
        this.ctx = this.gl = ctx;
        let vs = createShader(ctx, ctx.VERTEX_SHADER, vertSrc),
            fs = createShader(ctx, ctx.FRAGMENT_SHADER, fragSrc);
        this.shaders = { vs, fs };
        let prog = this.prog = this.program = createProgram(ctx, vs, fs);
        const getCurrentVars = (varsType, aou = varsType === ctx.ACTIVE_ATTRIBUTES? "Attrib": "Uniform") =>
            [...Array(ctx.getProgramParameter(prog, varsType))]
            .map((_, i) => {
                const {size, type, name} = ctx["getActive" + aou](prog, i),
                      loc                = ctx[`get${aou}Location`](prog, name);
                return {size, type, name: name.split("[")[0], loc};
       
