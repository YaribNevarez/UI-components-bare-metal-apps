# UI-components bare metal applications

The UI components library basically provides high level software classes for graphical user interface, there are three components: Frame Panel, Plot 2D, and text label. All UI components extends a base class Widget, this base class defines the basic interface for the extended components. The extended components basically utilize the software design patterns of factory, and composed (class extension), encapsulation and polymorphism. It can be created any number of UI components as needed, and they can be treated as base widgets having their own behaviour (polymorphism). The code is implemented in C++ style and uses dynamic memory allocation for the component instantiation.

# These UI components have been tested on ZYNQ device and require "Graphics-controller-ILI9341-ZYNQ" (driver layer)

# Widget

The widget class serves as the base UI component class, and it defines the basic component interface. It can be said that all and every UI component is a widget. This is an abstract class and cannot be instantiated.
```C
/*
 * widget.h
 *
 *  Created on: 9 de ene. de 2017
 *      Author: Yarib Nevárez
 */

#ifndef SRC_UI_COMPONENTS_WIDGET_H_
#define SRC_UI_COMPONENTS_WIDGET_H_

#include "xil_types.h"

typedef struct Widget_public Widget;

typedef void (* WidgetDrawFunc)    (Widget * obj);
typedef void (* WidgetRefreshFunc) (Widget * obj);
typedef void (* WidgetClearFunc)   (Widget * obj);
typedef void (* WidgetDeleteFunc)  (Widget ** obj);

#define WIDGET_VIRTUALTABLE_MEMBERS \
        WidgetDrawFunc    draw; \
        WidgetRefreshFunc refresh; \
        WidgetClearFunc   clear; \
        WidgetDeleteFunc  delete; \
        Widget *          parent; \
        Widget *          prev; \
        Widget *          next; \
        Widget *          first_child;


struct Widget_public
{
    WIDGET_VIRTUALTABLE_MEMBERS
};

#endif /* SRC_UI_COMPONENTS_WIDGET_H_ */
```

# Plot 2D

The Plot 2D is a widget intended to display data values in a graphical manner, the data is inserted as a dot and it will be displayed in the plot, then the following added dots will be displayed and connected by a line giving the effect of a line graph. This class is defined in plot2d.h.
```C
/*
 * plot2d.h
 *
 *  Created on: 3 de ene. de 2017
 *      Author: Yarib Nevárez
 */

#ifndef PLOT2D_H_
#define PLOT2D_H_

#include "xil_types.h"
#include "widget.h"

typedef struct Plot2D_public Plot2D;

#define PLOT2D_VIRTUALTABLE_MEMBERS \
        WIDGET_VIRTUALTABLE_MEMBERS \
        void     (* add_point) (Plot2D * obj, uint16_t value, uint32_t color);

struct Plot2D_public
{
    PLOT2D_VIRTUALTABLE_MEMBERS
};

Plot2D * Plot2D_new(uint16_t x, uint16_t y, uint16_t width, uint16_t height, uint32_t line_color, uint32_t background_color);

#endif /* PLOT2D_H_ */

The important function members of this class are the draw function and add_point. These functions are exhibited below.
static void Plot2D_draw(Plot2D * obj)
{
    if (obj != NULL)
    {
        Plot2D_private * thiz = (Plot2D_private *) obj;
        TFTGraphics *    canvas = thiz->canvas;
        if ((thiz->plot2D_space != NULL) && (canvas != NULL))
        {
            int16_t i;

            canvas->drawRectFilled(thiz->x_pos + 1, thiz->y_pos + 1, thiz->size,
                                   thiz->height, thiz->background_color);

            canvas->drawRect(thiz->x_pos, thiz->y_pos, thiz->size + 1, thiz->height + 1,
                             thiz->line_color);

            for (i = 0; i < thiz->index - 1; i ++)
            {
                canvas->drawLine(thiz->x_pos + i + 1,
                         thiz->y_pos + thiz->height - thiz->plot2D_space[i].y,
                         thiz->x_pos + i + 1 + 1,
                         thiz->y_pos + thiz->height - thiz->plot2D_space[i+1].y,
                         thiz->plot2D_space[i].c);
            }

            memcpy(thiz->plot2D_old_space,
                   thiz->plot2D_space,
                   sizeof(Point2D) * thiz->index);

            thiz->old_index = thiz->index;
        }
    }
}

static void Plot2D_add(Plot2D * obj, uint16_t y, uint32_t c)
{
    if (obj != NULL)
    {
        Plot2D_private * thiz = (Plot2D_private *) obj;
        if(thiz->plot2D_space != NULL)
        {
            if (thiz->height <= y)
            {
                y = 0;
                c = thiz->background_color;
            }

            if (thiz->index < thiz->size)
            {
                thiz->plot2D_space[thiz->index].y = y;
                thiz->plot2D_space[thiz->index].c = c;

                thiz->index ++;
            }
            else
            {
                memcpy(thiz->plot2D_space,
                       thiz->plot2D_space + 1,
                       sizeof(Point2D)*(thiz->size - 1));

                thiz->plot2D_space[thiz->size - 1].y = y;
                thiz->plot2D_space[thiz->size - 1].c = c;
            }
            Plot2D_refresh(obj);
        }
    }
}
```


# Text label

The text label is a widget intended to display text, it provides the functionality to set text and back ground colours, size and clean refresh of its graphic content. This class is defined in the textlabel.h.
The next lines of code shows the class definition.

```C
/*
 * textlabel.h
 *
 *  Created on: 4 de ene. de 2017
 *      Author: Yarib Nevárez
 */

#ifndef TEXTLABEL_H_
#define TEXTLABEL_H_

#include "xil_types.h"
#include "widget.h"

typedef struct TextLabel_public TextLabel;

typedef enum
{
    TEXT,
    NUMBER
} TextLabel_Type;

#define TEXTLABEL_VIRTUALTABLE_MEMBERS \
        WIDGET_VIRTUALTABLE_MEMBERS \
        void        (* set_x)     (TextLabel * obj, int32_t x_pos); \
        void        (* set_y)     (TextLabel * obj, int32_t y_pos); \
        void        (* set_width) (TextLabel * obj, int32_t width); \
        void        (* set_height)(TextLabel * obj, int32_t height); \
        void        (* set_text)  (TextLabel * obj, char * str); \
        void        (* set_number)(TextLabel * obj, int32_t number); \
        void        (* set_label_type)      (TextLabel * obj, TextLabel_Type type); \
        void        (* set_font_size)       (TextLabel * obj, uint8_t font_size); \
        void        (* set_text_color)      (TextLabel * obj, int32_t text_color); \
        void        (* set_background_color)(TextLabel * obj, int32_t background_color);

struct TextLabel_public
{
    TEXTLABEL_VIRTUALTABLE_MEMBERS
};

TextLabel * TextLabel_new (uint16_t x, uint16_t y, uint16_t width, uint16_t height, char * str, uint32_t text_color, uint32_t background_color);


#endif /* TEXTLABEL_H_ */
```

# Frame panel

The frame panel is a widget container, it holds other widgets as children in an internal tree data structure, and every leaf is a widget (Plot 2D, Text label, or even a Frame Panel). The widget tree can grow as bigger as needed. The Frame panel also displays border and background colours. The frame panel class is defined in framepanel.h.

```C
/*
 * framepanel.h
 *
 *  Created on: 10 de ene. del 2017
 *      Author: Yarib Nevárez
 */

#ifndef SRC_UI_COMPONENTS_FRAMEPANEL_H_
#define SRC_UI_COMPONENTS_FRAMEPANEL_H_

#include "xil_types.h"
#include "widget.h"

#define MAX_HEIGHT  320
#define MAX_WIDTH   240

typedef struct FramePanel_public FramePanel;

#define FRAMEPANEL_VIRTUALTABLE_MEMBERS \
        WIDGET_VIRTUALTABLE_MEMBERS \
        void (* set_x)     (FramePanel * obj, int32_t x_pos); \
        void (* set_y)     (FramePanel * obj, int32_t y_pos); \
        void (* set_width) (FramePanel * obj, int32_t width); \
        void (* set_height)(FramePanel * obj, int32_t height); \
        void (* set_lineColor)      (FramePanel * obj, int32_t text_color); \
        void (* set_backgroundColor)(FramePanel * obj, int32_t background_color);\
        void (* give_widget)        (FramePanel * obj, Widget * child);

struct FramePanel_public
{
    FRAMEPANEL_VIRTUALTABLE_MEMBERS
};

FramePanel * FramePanel_new (uint16_t x, uint16_t y, uint16_t width, uint16_t height, uint32_t line_color, uint32_t background_color);


#endif /* SRC_UI_COMPONENTS_FRAMEPANEL_H_ */
```

The following lines of code revels the easiness in the usage of UI components in the POXI application. This code shows the instantiation and deletion of the UI components.

```C
typedef struct
{
    FramePanel * framePanel;
    TextLabel  * statusLabel;
    TextLabel  * hbrLabel;
    Plot2D     * tracePlot;
} UIContent;

static UIContent Poxi_UI;


#define UI_LABEL_QTY (6)
static uint8_t Poxi_initUIContent(void)
{
    FramePanel * frame = FramePanel_new(0, 0, MAX_WIDTH -1, MAX_HEIGHT -1, WHITE, NAVY);
    TextLabel  * statusLabel = TextLabel_new(55, 80, 0, 0, "Starting Up", PINK, NAVY);
    TextLabel  * hbrLabel = TextLabel_new(140, 90, 0, 0, "-", WHITE, NAVY);
    Plot2D     * plot = Plot2D_new(30, 150, 100, 100, RED, BLACK);
    TextLabel  * label[UI_LABEL_QTY];
    uint8_t      i;
    uint8_t      rc;

    label[0] = TextLabel_new(25, 20, 0, 0, " University Bremerhaven        ", YELLOW, BLACK);
    label[1] = TextLabel_new(25, 30, 0, 0, " M.Sc. Embedded Systems Design ", WHITE, BLACK);
    label[2] = TextLabel_new(25, 40, 0, 0, " Pulse Oximeter                ", WHITE, BLACK);
    label[3] = TextLabel_new(25, 50, 0, 0, " March 2017                    ", WHITE, BLACK);
    
    label[4] = TextLabel_new(10, 80, 0, 0, "Status:", YELLOW, NAVY);
    label[5] = TextLabel_new(10, 90, 0, 0, "Heartbeat rate (bpm):", YELLOW, NAVY);

    rc = frame != NULL
      && statusLabel != NULL
      && hbrLabel != NULL
      && plot != NULL;

    for (i = 0; rc && i < sizeof(label)/sizeof(TextLabel *); i++)
        rc = rc && label[i] != NULL;

    if (rc)
    {
        frame->give_widget(frame, (Widget *)statusLabel);
        frame->give_widget(frame, (Widget *)plot);
        frame->give_widget(frame, (Widget *)hbrLabel);
        hbrLabel->set_label_type(hbrLabel, NUMBER);

        for (i = 0; i < sizeof(label)/sizeof(TextLabel *); i++)
            frame->give_widget(frame, (Widget *)label[i]);

        Poxi_UI.framePanel  = frame;
        Poxi_UI.statusLabel = statusLabel;
        Poxi_UI.tracePlot   = plot;
        Poxi_UI.hbrLabel    = hbrLabel;
    }

    return rc;
}

static uint8_t Poxi_run(void)
{
    Poxi_UI.framePanel->draw((Widget *)Poxi_UI.framePanel);

    Poxi_interruptDutyEnable = 1;
    do
    {
        Poxi_refreshUI();
        if (Poxi_ZYBO->switch_(0))
        {
            Poxi_JavaGUI_report();
        }
    } while (Poxi_ZYBO->switch_(3));
    Poxi_interruptDutyEnable = 0;

    Poxi_clip->light(LIGHTPROBE_OFF);
    Poxi_ZYBO->leds(0);

    Poxi_UI.statusLabel->set_text(Poxi_UI.statusLabel,"* STOP *");

    return 0;
}

static void Poxi_refreshUI(int variable)
{
    uint8_t hbr;

    if (Poxi_sampleStatus == UNAVAILABLE)
    {
        Poxi_UI.statusLabel->set_text_color(Poxi_UI.statusLabel, RED);
        Poxi_UI.statusLabel->set_text(Poxi_UI.statusLabel,"Sampling signal");
    }

    if (!Poxi_ZYBO->switch_(0))
    {
        Poxi_UI.tracePlot->add_point(Poxi_UI.tracePlot, variable*10, CYAN);
    }


    hbr = Poxi_DSP.ired->getStatistics(Poxi_DSP.ired).fundamentalFrec * 60;
    if (hbr <= 120)
    {
        if (hbr < 50)
            Poxi_UI.hbrLabel->set_text_color(Poxi_UI.hbrLabel, RED);
        else if (90 < hbr)
            Poxi_UI.hbrLabel->set_text_color(Poxi_UI.hbrLabel, MAGENTA);
        else
            Poxi_UI.hbrLabel->set_text_color(Poxi_UI.hbrLabel, WHITE);

        Poxi_UI.hbrLabel->set_number(Poxi_UI.hbrLabel, hbr);
    }
}

static void Poxi_dispose(void)
{
    //More code goes here
    if (Poxi_UI.framePanel != NULL)
        Poxi_UI.framePanel->delete((Widget **) &Poxi_UI.framePanel);
}
```
