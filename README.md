Now let's convert the existing `#capture` div into a Canvas and then export it as a PNG image using **native JavaScript + SVG foreignObject**, without any external library.

Add this script before `</body>` in your HTML file.

```

async function convertDivToImage() {
    const element = document.querySelector("#capture");

    // 1. Get element dimensions
    const rect = element.getBoundingClientRect();

    // 2. Clone the existing HTML element
    const clone = element.cloneNode(true);

    // 3. Copy computed styles
    function copyStyles(source, target) {
        const computed = window.getComputedStyle(source);

        for (const property of computed) {
            target.style.setProperty(
                property,
                computed.getPropertyValue(property)
            );
        }

        for (let i = 0; i < source.children.length; i++) {
            copyStyles(source.children[i], target.children[i]);
        }
    }

    copyStyles(element, clone);

    // 4. Serialize HTML
    const html = new XMLSerializer().serializeToString(clone);

    // 5. Create SVG containing HTML
    const svg = `
        <svg xmlns="http://www.w3.org/2000/svg"
             xmlns:xhtml="http://www.w3.org/1999/xhtml"
             width="${rect.width}"
             height="${rect.height}">

            <foreignObject width="100%" height="100%">
                ${html}
            </foreignObject>

        </svg>
    `;

    // 6. Convert SVG to Blob
    const blob = new Blob([svg], {
        type: "image/svg+xml;charset=utf-8"
    });

    const url = URL.createObjectURL(blob);

    try {
        // 7. Load SVG as Image
        const img = new Image();
        img.src = url;

        await img.decode();

        // 8. Draw Image onto Canvas
        const canvas = document.createElement("canvas");

        const scale = 2;

        canvas.width = rect.width * scale;
        canvas.height = rect.height * scale;

        const ctx = canvas.getContext("2d");

        ctx.scale(scale, scale);
        ctx.drawImage(img, 0, 0);

        // 9. Convert Canvas to PNG
        const image = canvas.toDataURL("image/png");

        // 10. Display generated image
        const preview = document.createElement("img");
        preview.src = image;
        preview.style.maxWidth = "100%";

        document.body.appendChild(preview);

        return image;

    } finally {
        URL.revokeObjectURL(url);
    }
}

convertDivToImage();

```

### **Conversion flow**

**Existing HTML div**

\#capture

**Clone DOM + computed styles**

**SVG foreignObject**

**Image object**

**Canvas → PNG image**

**Important:** This is a basic implementation. Complex CSS, pseudo-elements, external images, and form-control states may need additional handling. Also, SVG `foreignObject` support and rendering can vary.
