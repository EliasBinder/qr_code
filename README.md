# QRCode

[![Continuous Integration](https://github.com/iodevs/qr_code/workflows/Continuous%20Integration/badge.svg)](https://github.com/iodevs/qr_code/actions)
[![Coverage Status](https://coveralls.io/repos/github/iodevs/qr_code/badge.svg?branch=master)](https://coveralls.io/github/iodevs/qr_code?branch=master)
![GitHub top language](https://img.shields.io/github/languages/top/iodevs/qr_code)
![Hex.pm](https://img.shields.io/hexpm/v/qr_code)
![Hex.pm](https://img.shields.io/hexpm/dt/qr_code)

This library is useful for generating QR code to your projects.

![QR code](docs/qrcode.svg)

## Installation

```elixir
def deps do
  [
    {:qr_code, "~> 3.1.1"}
  ]
end
```

## Usage

If you want to create QR code, just use the function `QRCode.create(your_string, level)`:

```elixir
  iex> QRCode.create("Hello World")
  {:ok, %QRCode.QR{...}}
```

You can also change the error correction level according to your needs. There are four level of error corrections:

```elixir
  | Error Correction Level    | Error Correction Capability    |
  |---------------------------|--------------------------------|
  | :low (default value)      | recovers 7% of data            |
  | :medium                   | recovers 15% of data           |
  | :quartile                 | recovers 25% of data           |
  | :high                     | recovers 30% of data           |
```

> Be aware higher levels of error correction require more bytes, so the higher the error correction level,
> the larger the QR code will have to be.

We've just generated QR code and now we want to save it to some image file format with high quality. This library supports two image types: `svg` and `png`. So basic using is following:

```elixir
  iex> "Hello World"
        |> QRCode.create(:high)
        |> QRCode.render()
        |> QRCode.save("/path/to/hello.svg")
  {:ok, "/path/to/hello.svg"}
```

where we used an error correction level `:high`. Note the function `QRCode.render()` has default render set up to `:svg` with default `SvgSettings`. Similarly, if you want to export to png just use `QRCode.render(:png)` with/without `PngSettings`. Calling by `QRCode.save` function you save QR code to file `/path/to/file.type`. Also instead of saving QR code to file, you can use a function `QRCode.to_base64()` to encode QR to base 64.

There are several options to change the appearance of svg or png:

### Svg settings

```elixir
| Option             | Type                   | Default value | Description                         |
|--------------------|------------------------|---------------|-------------------------------------|
| scale              | positive integer       | 10            | changes size of rendered QR code    |
| background_opacity | nil or 0.0 <= x <= 1.0 | nil           | sets background opacity of svg      |
| background_color   | string or {r, g, b}    | "#ffffff"     | sets background color of svg        |
| qrcode_color       | string or {r, g, b}    | "#000000"     | sets color of QR                    |
| flatten            | boolean                | false         | renders path data instead of rects  |                                     |
| image              | {string, size} or nil  | nil           | puts the image to the center of svg |
| structure          | :minify or :readable   | :minify       | minifies or makes readable svg file |
```

Notes:

- `:image` inserts image `/path/to/image.type` with `size`, this number must be positive integer.
  There are a few limitations:

  - The only image formats SVG software must support are JPEG, PNG, and other SVG files, see [MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/image).
  - Pay attention to the `size` of the embedded image, if you put it too large, it may not be readable by the QR reader.

- By `:structure` you can minify a final size of svg file or make it readable if you need. In the readable case, the file size can be slightly larger and the svg code is structured and thus more clearer.

Let's see an example with embedded image below:

```elixir
  iex> alias QRCode.Render.SvgSettings
  iex> image = {"/docs/elixir.svg", 100}
  iex> qr_color = {17, 170, 136}
  iex> svg_settings = %SvgSettings{qrcode_color: qr_color, image: image, structure: :readable}
  %QRCode.Render.SvgSettings{
    background_color: "#ffffff",
    background_opacity: nil,
    image: {"/docs/elixir.svg", 100},
    qrcode_color: {17, 170, 136},
    scale: 10,
    structure: :readable
  }
  iex> "your_string"
        |> QRCode.create()
        |> QRCode.render(:svg, svg_settings)
        |> QRCode.save("/tmp/qr-with-image.svg")
  {:ok, "/tmp/qr-with-image.svg"}
```

![QR code color](docs/qrcode_color_with_image.svg)

Similarly, you can use `png` with/without `PngSettings`.

### Png settings

```elixir
| Option            | Type                   | Default value | Description                  |
|--------------------|------------------------|---------------|------------------------------|
| scale              | positive integer       | 10            | changes size of rendered QR  |
| background_color   | string or {r, g, b}    | "#ffffff"     | sets background color of png |
| qrcode_color       | string or {r, g, b}    | "#000000"     | sets color of QR             |
```

## Limitations

The QR code is limited by characters that can contain. In our case this library was developed only for `Byte` and `Alphanumeric` mode. For example, the limits for **40 version** are:

```elixir
|                   |      Maximum number of characters      |
| Level             |   low   |  medium  | quartile |  high  |
|-------------------|---------|----------|----------|--------|
| Byte mode         |   2953  |   2331   |   1663   |  1273  |
| Alphanumeric mode |   4296  |   3391   |   2420   |  1852  |
```

For other versions and modes see [Character Capacities](https://www.thonky.com/qr-code-tutorial/character-capacities) in documentation.

If you get an `{:error, "Input string can't be encoded"}` it means that you overcame the limit for the given `version` and `level` (i.e. your string is too big).

If anyone needs the rest of encoding modes,
please open new issue or push your code in this repository.

## Notes

- You can also save the QR matrix to csv using by [csvlixir](https://github.com/jimm/csvlixir):

  ```elixir
  {:ok, qr} = QRCode.create("Hello World")
  save_csv(qr.matrix, "qr_matrix.csv")

  def save_csv(matrix, name_file) do
    name_file
    |> File.open([:write], fn file ->
      matrix
      |> CSVLixir.write()
      |> Enum.each(&IO.write(file, &1))
    end)
  end
  ```

## References

- [http://www.thonky.com/qr-code-tutorial/](http://www.thonky.com/qr-code-tutorial/)

## License

QRCode source code is licensed under the _BSD-4-Clause._

---

Created: 2018-11-24Z
